# Deployment and Operations

Career OS runs as Docker containers on a single host the user controls, with a path to Kubernetes if the user wants to practise it (this repository is a DevOps playground after all). The constraint is that one person operates it in spare time, so defaults are conservative and automated.

## Target environments

| Environment | Purpose | Shape |
|---|---|---|
| Local development | Day-to-day coding | `pnpm dev` for web and worker against a Postgres container; hot reload |
| Preview (optional) | Try a branch | Same Compose file with a different project name and port |
| Production (self-hosted) | Daily use | Docker Compose on a home server, NAS, or small VPS; private network exposure |
| Kubernetes (later) | Learning and resilience | Helm chart in `infra/k8s`; CloudNativePG or managed Postgres |

## Docker Compose topology

```yaml
# infra/compose/docker-compose.yml (shape, not final)
services:
  caddy:
    image: caddy:2
    ports: ["443:443", "80:80"]
    volumes: [./Caddyfile:/etc/caddy/Caddyfile, caddy_data:/data]
    depends_on: [web]
  web:
    image: ghcr.io/<owner>/career-os-web:<tag>
    env_file: .env
    environment:
      DATABASE_URL: postgres://careeros:${POSTGRES_PASSWORD}@postgres:5432/careeros
      ATTACHMENTS_DIR: /data/attachments
    volumes: [attachments:/data/attachments]
    depends_on: { postgres: { condition: service_healthy } }
    healthcheck: { test: ["CMD", "wget", "-qO-", "http://localhost:3000/api/v1/health"], interval: 30s }
  worker:
    image: ghcr.io/<owner>/career-os-worker:<tag>
    env_file: .env
    environment: { DATABASE_URL: ..., ATTACHMENTS_DIR: /data/attachments }
    volumes: [attachments:/data/attachments]
    depends_on: { postgres: { condition: service_healthy } }
  postgres:
    image: postgres:17
    environment: { POSTGRES_USER: careeros, POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}, POSTGRES_DB: careeros }
    volumes: [pgdata:/var/lib/postgresql/data]
    healthcheck: { test: ["CMD-SHELL", "pg_isready -U careeros"], interval: 10s }
  backup:
    image: ghcr.io/<owner>/career-os-backup:<tag>   # pg_dump + tar + optional rclone, cron inside
    env_file: .env
    volumes: [attachments:/data/attachments:ro, backups:/backups]
    depends_on: [postgres]
volumes: { pgdata: {}, attachments: {}, backups: {}, caddy_data: {} }
```

Optional profiles: `observability` (Grafana, Loki, Tempo, OpenTelemetry Collector) and `mail` (an SMTP relay) for later features.

## Configuration

All configuration is environment variables loaded and validated by `packages/config`; runtime-tunable values live in the `settings` table.

| Variable | Required | Purpose |
|---|---|---|
| `DATABASE_URL` | yes | Postgres connection |
| `ANTHROPIC_API_KEY` | yes | LLM access (Docker secret `anthropic_api_key` also supported) |
| `APP_ORIGIN` | yes | Public origin for cookies and CSRF checks, e.g. `https://careeros.tail1234.ts.net` |
| `SESSION_SECRET` | yes | Cookie signing key |
| `ATTACHMENTS_DIR` | yes | Attachment storage path |
| `TZ_DEFAULT` | no | Fallback timezone before the user sets one |
| `LOG_LEVEL` | no | `info` default |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | no | Enables tracing and metrics export when set |
| `BACKUP_TARGET`, `BACKUP_ENCRYPTION_KEY_FILE` | no | Off-host backup destination and encryption |
| `SMTP_URL`, `PUSH_URL` | no | Later notification channels |

Images are built once and configured at runtime; no environment-specific builds.

## Build and release

- GitHub Actions on every pull request: install, lint, typecheck, unit and integration tests (Testcontainers), contract drift check, Playwright smoke; on `main`: build multi-arch images (amd64, arm64) for `web`, `worker`, `backup`, push to GHCR with `sha` and `latest` tags; on a version tag: also tag the version and attach a changelog.
- Database migrations run as a one-shot step by the `web` container on start (`drizzle-kit migrate`) with an advisory lock so web and worker do not race. Migrations are forward-only; rollbacks are new migrations.
- Upgrade procedure: `docker compose pull && docker compose up -d`; the release notes state when a backup should be taken first (always before a major version).

## Network exposure

Recommended: keep the host on a private overlay network (Tailscale or WireGuard) and access the app through its overlay hostname with TLS from Caddy using the overlay's certificate integration or a private CA. If public exposure is needed (for example phone access without a VPN client), use a real domain with automatic TLS and keep the login throttling and TOTP enabled. ADR-0007 records this.

## Observability

| Signal | v1 default | Optional profile |
|---|---|---|
| Logs | pino JSON to stdout, `docker compose logs`; request ID, session ID correlation | Loki via Promtail or the OpenTelemetry Collector |
| Metrics | `/api/v1/health` with checks; usage and cost in the database and UI | Prometheus-style metrics through the OpenTelemetry SDK: request latency, job durations, LLM tokens and cost, cache ratio, job failures |
| Traces | Off | OpenTelemetry traces across route handler → domain → database and LLM calls |
| Alerts | In-app notifications for job failures, backup failures, budget | Grafana alerting if the profile is enabled |

Health semantics: `web` is ready when the database responds and migrations are current; `worker` writes a heartbeat row every minute and `health` reports `worker: stale` if older than five minutes.

## Backup and restore

- Nightly `pg_dump --format=custom` plus a tar of the attachments volume, both into `backups/` with date stamps; 30-day local retention.
- Optional off-host copy with rclone to object storage or another machine, encrypted with age; the key lives in a password manager, not on the host.
- Weekly automated restore check inside the backup container: restore the latest dump into a scratch database, run a row-count sanity query, report to `backup_runs`.
- Manual restore runbook: stop web and worker, restore the dump into a fresh volume, restore attachments, start services, sign in, verify the dashboard. Rehearsed once per release (constitution IX).

## Runbook (summary)

| Situation | Action |
|---|---|
| App not reachable | `docker compose ps`; check Caddy logs; check `web` health |
| Agent turns failing | Open the session record for the reason; check API key validity and the provider status; check budget state |
| Worker stale | `docker compose logs worker`; restart; pg-boss resumes pending jobs |
| Budget exhausted | Raise the budget in settings or wait for the month to roll; background jobs resume automatically |
| Disk filling | Prune old backups and Docker images; attachments quota in settings |
| Host migration | Export or restore from the latest backup on the new host; update `APP_ORIGIN` |
| API key rotation | Replace secret, `docker compose up -d web worker` |

## Kubernetes path (later)

One Deployment each for `web` and `worker`, a CronJob for backups, a PostgreSQL cluster via CloudNativePG (or a managed instance), an Ingress with TLS, a Secret for the API key and session secret, a PersistentVolumeClaim for attachments (or object storage behind the same storage interface), and ConfigMaps for non-secret configuration. A Helm chart under `infra/k8s` mirrors the Compose file. No code changes are expected: configuration is environment-driven and both processes are stateless apart from the database and the attachments volume.
