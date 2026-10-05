# ADR-0010: Docker Compose first, Kubernetes later

**Status:** Accepted · **Date:** 2026-10-05

## Context

The user chose self-hosted containers and wants a path to Kubernetes, in a repository named for DevOps practice. The operator is one person; availability requirements are low; data must stay on controlled infrastructure.

## Decision

Ship **Docker Compose** as the supported production deployment for v1: `caddy`, `web`, `worker`, `postgres`, `backup`, with optional `observability` and `mail` profiles. Images are multi-arch and published to GHCR by CI. Keep both processes stateless apart from PostgreSQL and the attachments volume so a **Helm chart** can be added later under `infra/k8s` without code changes.

## Alternatives considered

- **Kubernetes from day one.** Valuable practice, but a cluster to run for one user doubles the operational load before the product exists. Better as a deliberate second deployment target once the app is stable.
- **Managed serverless (Vercel plus managed Postgres).** Least operations, but the user chose to keep data on their own infrastructure, and the worker's long-running jobs fit a container better than function limits.
- **Bare-metal systemd services.** Works, but containers give reproducible images and the same artefacts for Compose and Kubernetes.

## Consequences

- One `docker compose up` deployment; a documented runbook; backups as a sidecar.
- Kubernetes migration is a packaging exercise: Deployments, CronJob, PostgreSQL operator or managed instance, Ingress, Secrets, PVC.
- Hosting on a small ARM board or a VPS both work thanks to multi-arch images.

## Revisit triggers

- The user wants to practise Kubernetes operations (add the chart as feature work, keep Compose supported).
- A requirement for high availability appears (unlikely for one user).
