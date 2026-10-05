# Feature Specification: Secure Access and Settings

**Feature Branch**: `001-secure-access-settings`

**Created**: 2026-10-05

**Status**: Draft

**Input**: User description: "Build the foundation that lets a single person use Career OS privately. On first launch, the person creates the one and only account with a strong password. Afterwards they sign in and stay signed in across browser restarts for a configurable period. Repeated failed sign-ins are throttled and logged. Signed in, the person manages settings: timezone, week start day, preferred check-in times, which model the assistant uses and how much effort it spends per kind of task, a monthly spending budget for the assistant, and how autonomous the assistant may be (proposals only by default). A usage page shows assistant spend for the current month against the budget, with warnings at 80 percent and a pause of background assistant work at 100 percent. Health of the system (database reachable, worker alive, last backup time) is visible on the same page. Why: everything else depends on a trusted private space and a known cost ceiling."

## User Scenarios & Testing *(mandatory)*

Career OS is used by exactly one person, called "the person" below, on a computer they control. The stories are ordered by how much the rest of the product depends on them. Each story can be built, tested, and shown on its own.

### User Story 1 - Create the one and only account on first launch (Priority: P1)

The first time the person opens Career OS, no account exists. The application guides them to create the single account: an email address used as the sign-in name, a display name, and a strong password. Once the account exists, the person is signed in and asked to confirm their timezone and week start day, so that everything that follows is dated correctly. This story includes storing those two settings; story 4 covers the full settings screen. No second account can ever be created.

**Why this priority**: Without an account nothing in the product is private. Every other feature assumes a signed-in person. This is the first step of the first-run journey.

**Independent Test**: Start the application with no data, open it in a browser, and complete the account creation screen. This delivers a private, password-protected space. Then confirm that a second visit to the account creation screen is refused.

**Acceptance Scenarios**:

1. **Given** no account exists, **When** the person opens the application, **Then** they are guided to create the single account and no other screen is reachable until the account exists.
2. **Given** the account creation screen, **When** the person enters a password shorter than 12 characters, **Then** the password is rejected with a message stating the minimum length, and nothing is created.
3. **Given** the account creation screen, **When** the person enters a valid email address, a display name, and a password of at least 12 characters, **Then** the account is created, the person is signed in, and they are asked to confirm their timezone and week start day, pre-filled from the browser where possible.
4. **Given** an account exists, **When** anyone opens the account creation screen or submits an account creation request, **Then** the request is refused, no account is changed, and the attempt is recorded in the sign-in activity.

---

### User Story 2 - Sign in and stay signed in (Priority: P1)

The person signs in with their email address and password. They stay signed in across browser restarts and across restarts of the application itself, until they sign out or have not used the application for a configurable period (30 days by default). They can see which browsers are currently signed in and can sign out of all of them at once.

**Why this priority**: Equal in importance to story 1. Daily use happens on a laptop at work and on a phone in between; being asked for the password every day would stop the five-minute check-in habit from forming.

**Independent Test**: With an account present, sign in, close the browser, reopen it within the signed-in period, and confirm that no sign-in prompt appears. Then sign out and confirm that every page asks for sign-in again.

**Acceptance Scenarios**:

1. **Given** an account exists and the person is not signed in, **When** they open any page, **Then** they see the sign-in screen and, after entering the correct password, are taken to the page they asked for.
2. **Given** the person is signed in, **When** they close the browser and return within the signed-in period, **Then** they remain signed in without re-entering the password.
3. **Given** the person is signed in, **When** the application is restarted, **Then** the person remains signed in.
4. **Given** the person has not used the application for longer than the signed-in period, **When** they return, **Then** they must sign in again.
5. **Given** the person is signed in on two browsers, **When** they choose "sign out everywhere" on one of them, **Then** both browsers require sign-in on their next use.
6. **Given** the person is signed in, **When** they change the signed-in period to 7 days, **Then** every session from then on ends after 7 days without use.

---

### User Story 3 - Throttled and logged failed sign-ins (Priority: P1)

Wrong passwords are slowed down and recorded. After five failures within ten minutes, every further attempt must wait before it is checked: 2 seconds at first, doubling with each further failure, never more than 5 minutes. After twenty consecutive failures, sign-in is locked for 60 minutes. Every attempt, successful or not, is recorded so that the person can notice unexpected activity.

**Why this priority**: The application may be reachable by other devices on the person's network. Guessing the single password must be slow enough to be hopeless and visible enough to be noticed.

**Independent Test**: Submit wrong passwords repeatedly. Observe that the sixth attempt within ten minutes is checked only after a 2-second wait, the seventh after a 4-second wait, the twenty-first is refused outright, and the sign-in activity view lists every attempt with its outcome and, where it waited, how long it waited.

**Acceptance Scenarios**:

1. **Given** an account exists, **When** a wrong password is entered five times within ten minutes, **Then** each further attempt is checked only after a wait that starts at 2 seconds and doubles with each further failure, the screen says that attempts are being delayed and shows the remaining wait, and each attempt is recorded once with its final outcome and how long it waited.
2. **Given** twenty consecutive failed attempts, **When** another attempt is made, **Then** it is refused without checking the password, the screen says when sign-in will be possible again, and the lock lifts after 60 minutes with the failure count unchanged, so one more failed attempt starts a new 60-minute lock.
3. **Given** any sign-in attempt, **When** it completes with any outcome, **Then** a record is kept with the time, the outcome, the source address, and a browser description, and the entered password is never stored.
4. **Given** failed attempts have been counted, **When** a correct password is accepted, **Then** the failure count returns to zero and the successful sign-in is recorded.
5. **Given** the person is signed in, **When** they open the sign-in activity view, **Then** they see the attempts of at least the last 30 days, newest first.

---

### User Story 4 - Set timezone, week start day, and check-in times (Priority: P1)

The person sets their timezone, the day their week starts, and their preferred check-in times, so that schedules match their life. Changes take effect at once: the settings screen shows the next occurrence of each check-in time in the person's timezone, and after any change that next occurrence is recomputed.

**Why this priority**: Every dated part of the product (weekly plans, check-ins, reminders, the month boundaries of the budget) must agree with the person's own calendar. The first-run journey sets these right after the password.

**Independent Test**: Change the timezone, the week start day, and each check-in time; reload; confirm the stored values. Confirm that an unrecognised timezone or an invalid time is rejected with a clear message.

**Acceptance Scenarios**:

1. **Given** the person is signed in, **When** they open settings, **Then** they see the current timezone, week start day, and check-in times, with the defaults shown if nothing was changed yet.
2. **Given** the settings screen, **When** the person changes the timezone to a recognised region and saves, **Then** the change is visible immediately, and the next occurrence shown for each check-in time is recomputed in the new timezone and never lies in the past.
3. **Given** the settings screen, **When** the person sets the daily check-in time to 08:00 and allows check-ins on weekends, **Then** the stored preference shows 08:00 every day.
4. **Given** the settings screen, **When** the person enters a timezone that is not recognised or a time that is not a valid time of day, **Then** the save is refused with a message naming the field, and no field is changed.

---

### User Story 5 - Configure the assistant: model, effort, budget, autonomy (Priority: P1)

The person decides which model the assistant uses, how much effort it spends on each kind of task (roadmap drafting, re-planning, weekly planning, link suggestions, check-ins, coaching), how much it may spend per month in USD, and whether it may apply the two low-risk operation types (plan item status and key result progress) without a separate approval. After installation the defaults are the product's default model, effort tuned per kind of task, 40 USD per month, and "proposals only".

**Why this priority**: The cost ceiling and the autonomy rule must exist before the first assistant feature ships. Otherwise the first roadmap draft could run without a limit or apply changes nobody approved.

**Independent Test**: Open the assistant settings, change each value, save, reload, and confirm the values. Confirm that autonomy is "proposals only" after a fresh installation and that switching it away requires an explicit confirmation.

**Acceptance Scenarios**:

1. **Given** a fresh installation, **When** the person opens the assistant settings, **Then** the model is the product default, each kind of task shows its default effort, the monthly budget is 40 USD, and autonomy is "proposals only".
2. **Given** the assistant settings, **When** the person picks another model from the offered list and saves, **Then** every assistant session started after the save uses the new model, and sessions already running keep the model they started with.
3. **Given** the assistant settings, **When** the person lowers the effort for check-ins to "low" and raises the effort for weekly planning to "very high", **Then** each kind keeps its own value and the other kinds are unchanged.
4. **Given** the assistant settings, **When** the person sets the monthly budget to 25 USD, **Then** the stored budget is 25 USD and the next budget check uses 25 USD as the ceiling.
5. **Given** autonomy is "proposals only", **When** the person switches to "auto-apply low-risk operations", **Then** they must confirm a plain explanation that names the two operation types that become automatic and the condition on them before the change is saved, and the change is recorded with its time.
6. **Given** the assistant settings, **When** the person enters a negative budget, an amount with more than two decimals, or an effort level outside the scale, **Then** the save is refused with a message naming the field, and nothing is changed.

---

### User Story 6 - See assistant spend against the budget, with warning and pause (Priority: P2)

A usage page shows what the assistant has cost this month against the monthly budget, broken down by kind of task, by day, and by session. When spend reaches 80 percent, the person is warned in the application. When spend reaches 100 percent, background assistant work pauses: tasks that become due are skipped with a notice, and starting an interactive session asks for confirmation. The pause lifts when the budget is raised or the month rolls over.

**Why this priority**: This is the "known cost ceiling". It is P2 only because there is no assistant spend to show until the assistant features exist. It must be in place before the first of them ships.

**Independent Test**: Insert test usage records that add up to 79, 80, and 100 percent of the budget, open the usage page after each step, and confirm the figures, the warning, and the pause. Confirm that a background task due during the pause does not run and that raising the budget lifts the pause. Until a real assistant task exists, use a stand-in background task that only asks the budget gate and records whether it ran or was skipped.

**Acceptance Scenarios**:

1. **Given** usage records exist for the current month, **When** the person opens the usage page, **Then** they see the month's spend in USD, the budget, the percentage used, the remaining amount, and breakdowns by kind of task, by day, and by session.
2. **Given** spend reaches 80 percent of the budget, **When** the usage page is opened, **Then** a warning is shown, and a budget warning notice appears at the top of the usage page and the settings screen no later than the start of the next day in their timezone.
3. **Given** spend reaches 100 percent of the budget, **When** a background assistant task becomes due, **Then** it is skipped, the skip is recorded with its reason, and a notice about the skipped task appears at the top of the usage page and the settings screen.
4. **Given** background work is paused, **When** the person starts an interactive assistant session, **Then** they are asked to confirm that they want to spend beyond the budget before the session starts.
5. **Given** background work is paused, **When** the person raises the budget above the month's spend, **Then** the pause lifts before the next background task is considered.
6. **Given** the month ends, **When** the first day of the new month starts in the person's timezone, **Then** spend for the new month starts at zero, the warning and pause states clear, and the previous month remains viewable.

---

### User Story 7 - See system health on the usage page (Priority: P2)

On the same usage page the person sees whether the database is reachable, whether the background worker is alive (its last report time, flagged as not reporting after five minutes of silence), when the last successful backup finished, and the outcome of the most recent backup run.

**Why this priority**: One person operates the system in spare time. A single glance must answer "is it running, and is my data safe" before anything else is trusted. The health section can be shown on its own if story 6 is not yet built, and shares the usage page once both exist.

**Independent Test**: Open the usage page with everything running. Stop the worker and reload after six minutes; confirm the worker shows as not reporting. With no backup run recorded, confirm the page says so without failing.

**Acceptance Scenarios**:

1. **Given** all parts are running, **When** the person opens the usage page, **Then** the health section shows the database as reachable, the worker as alive with its last report time, the time of the last successful backup, and the application's current time in the person's timezone.
2. **Given** the worker has not reported for more than five minutes, **When** the page is opened, **Then** the worker is shown as "not reporting since" its last report time.
3. **Given** no backup has ever been recorded, **When** the page is opened, **Then** the backup item says that no backup has been recorded yet, and the rest of the page works normally.
4. **Given** the most recent backup run failed, **When** the page is opened, **Then** the backup item shows the failure with its reason and still shows the time of the last successful backup before it.
5. **Given** the database cannot be reached, **When** the person opens the page, **Then** the application shows a plain message that the database is unreachable instead of a blank page or an unexplained error.

---

### User Story 8 - Change the password (Priority: P2)

Signed in, the person can change their password by entering the current password and a new one that meets the password rule. Afterwards every other signed-in browser must sign in again.

**Why this priority**: Needed for good hygiene and after any suspected exposure, but not on the first day.

**Independent Test**: Sign in on two browsers, change the password on one, confirm the other requires sign-in, and confirm the old password no longer works.

**Acceptance Scenarios**:

1. **Given** the person is signed in, **When** they enter the correct current password and a new password of at least 12 characters, **Then** the password is changed, the change is recorded, and every other session ends.
2. **Given** the password change screen, **When** the person enters a wrong current password, **Then** nothing changes, and the attempt counts and is recorded like a failed sign-in.
3. **Given** the password change screen, **When** the new password is shorter than 12 characters, **Then** it is rejected with a message stating the rule, and nothing changes.

---

### User Story 9 - Recover access to the single account (Priority: P1)

There is no second account and no administrator. If the person forgets the password or the sign-in email address, loses the device holding the second factor, or locks themselves out, they need a way back in that works only for someone who controls the computer the application runs on, and never for someone who merely reaches the application over the network.

**Why this priority**: A forgotten password or a lock on the only account would otherwise mean losing access to everything, and the lock from story 3 exists from the first release. The path is small, a documented step on the host with no screen, so it ships with the first release.

**Independent Test**: With the password unknown and sign-in locked, use the recovery path from the host, set a new password, and sign in. Confirm that the recovery path is not reachable from a browser on the network.

**Acceptance Scenarios**:

1. **Given** the password is forgotten, **When** the person uses the recovery path on the host, **Then** they can set a new password that meets the password rule, every existing session ends, and the recovery is recorded.
2. **Given** sign-in is locked after repeated failures, **When** the person uses the recovery path, **Then** the lock and the failure count are cleared.
3. **Given** the second factor is enabled and its device is lost, **When** the person uses the recovery path, **Then** they can turn the second factor off and sign in with the password alone. This scenario applies once story 10 is delivered.
4. **Given** the sign-in email address is forgotten, **When** the person uses the recovery path on the host, **Then** it shows the current sign-in email address and lets them set a new one.
5. **Given** someone has only network access to the application, **When** they look for the recovery path, **Then** it is not available to them.

---

### User Story 10 - Optional second sign-in factor (Priority: P3)

The person can add a second factor to sign-in: a one-time code generated by an authenticator app on their phone. When enabled, sign-in needs the password and then a current code. A set of one-time recovery codes, shown once, lets the person sign in if the phone is unavailable.

**Why this priority**: The recommended setup keeps the application on a private network, where the password and throttling are sufficient. The second factor matters if the application is ever reachable from the public internet, so it is offered but not required.

**Independent Test**: Enable the second factor, sign out, sign in with password and code, then sign in with a recovery code, then disable the factor. Confirm each step behaves as described.

**Acceptance Scenarios**:

1. **Given** the second factor is off, **When** the person enables it, **Then** they must confirm with a current code before it takes effect, and the application then shows ten recovery codes exactly once with a reminder to store them safely.
2. **Given** the second factor is on, **When** the person signs in with the correct password, **Then** they are asked for a current code, and a wrong code counts and is recorded like a failed sign-in.
3. **Given** the second factor is on, **When** the person signs in with the password and an unused recovery code, **Then** they are signed in, that code can never be used again, and they are told how many codes remain.
4. **Given** the second factor is on, **When** the person generates a new set of recovery codes, **Then** every earlier code stops working.
5. **Given** the second factor is on, **When** the person turns it off by entering the current password and a current code or recovery code, **Then** it is off, the change is recorded, and every other session ends.

---

### Edge Cases

- First run when an account already exists: the account creation screen is not shown, the sign-in screen appears instead, and any direct creation request is refused and recorded.
- Lockout while the only account is locked: new sign-ins are refused for 60 minutes and the screen says when the lock lifts. When the lock lifts the failure count is unchanged, so one more failed attempt starts a new 60-minute lock. Sessions that were already signed in keep working. If the person cannot wait, the recovery path on the host clears the lock and the count.
- An attempt submitted while an earlier attempt is still waiting under the throttling rule: it is refused without being checked, the screen shows the remaining wait, it is recorded with the outcome "refused", and it does not count as a failure.
- Budget lowered to or below the current month's spend: the warning or pause state applies immediately, before the next background task, and the usage page shows the new ceiling and the state.
- Budget set to 0 USD: background assistant work pauses at once and every interactive session asks for confirmation, which works as a switch to turn the assistant off. The usage page shows the state as paused and shows the percentage used as "not applicable" instead of a number or an error.
- Month rollover: at the start of the first day of the month in the person's timezone, month-to-date spend restarts at zero, warning and pause states clear, skipped tasks are not run retroactively, and the previous month stays viewable with its figures unchanged. A request is counted in the month in which it was made.
- Worker not reporting: after five minutes of silence the health section shows "not reporting since" the last report time; if it never reported, it says so. Interactive assistant sessions and the usage page do not depend on the worker.
- No backup ever taken: the health section says that no backup has been recorded yet, visibly flagged as a caution, and nothing else on the page breaks.
- Timezone change affecting scheduled times: the next occurrence of every configured check-in time, including the monthly and quarterly ones, is recomputed in the new timezone and shown on the settings screen. An occurrence whose configured time has already passed today in the new timezone moves to its next occurrence, never to the past and never to "now". Day and month boundaries for the budget move with the timezone.
- Invalid settings values: an unrecognised timezone, a week start that is not a day of the week, a time that is not a valid time of day, a budget that is negative, non-numeric, above the maximum, or has more than two decimals, an effort level outside the scale, an autonomy level outside the two allowed values, a monthly or quarterly rule outside the allowed values in FR-016, or a signed-in period outside 1 to 90 days is refused with a message naming the field and the allowed values, and nothing is saved from that request.
- Multiple browsers signed in: each browser has its own session with its own expiry; the sessions list shows all of them; "sign out everywhere", a password change, a second-factor change, and the recovery path end all of them; a setting changed in one browser is visible in the others on their next load; when two browsers save settings at the same time, the later save wins and both saves are recorded.
- Clock skew: all timing decisions (session expiry, throttling windows, lock duration, day and month boundaries, worker staleness) use the application's own clock, never the browser's. The usage page shows the application's current time so that a wrong host clock is visible. If the host clock drifts far enough that one-time codes are rejected, a recovery code or the recovery path still works.
- The chosen model is no longer offered after an update: the settings screen flags the stored choice, new assistant sessions use the product default until the person picks another model, and a notice (FR-048) is created.
- A request is made with a model that has no price in the price list, for example by a session that was already running when an update removed the model: the usage record is written with a cost of zero and flagged as unpriced, a notice (FR-048) is created, and the usage page shows how many unpriced requests the month has.
- A request fails part way: the usage record is still written with how the request ended, and its cost still counts toward the month.
- Database unreachable: every screen that can still render shows a plain "database unreachable" message; nothing silently shows stale data as current.

## Requirements *(mandatory)*

### Functional Requirements

*Account and first run*

- **FR-001**: System MUST support exactly one account. While no account exists, System MUST direct every visit to the account creation screen and MUST NOT show any other screen or data, except the operator signal in FR-010.
- **FR-002**: System MUST create the account from an email address (the sign-in name), a display name, and a password, MUST sign the person in immediately afterwards, and MUST then ask them to confirm their timezone and week start day before any other screen opens; accepting the pre-filled values with one action is enough, and the settings screen remains the place to change them later.
- **FR-003**: System MUST require a password of at least 12 characters whenever a password is set or changed, MUST state the rule before entry, and MUST reject a shorter password with a plain message. System MAY additionally reject a password that appears in a list of commonly used passwords held on the host; no password is ever sent outside the host for checking.
- **FR-004**: Once an account exists, System MUST refuse every further account creation attempt, MUST NOT show the creation screen, and MUST record each attempt in the sign-in attempt log.

*Sign-in and sessions*

- **FR-005**: System MUST let the person sign in with the email address and password and, when the second factor is enabled, a current one-time code or an unused recovery code.
- **FR-006**: System MUST keep the person signed in across browser restarts and application restarts until one of these happens: they sign out, the session is unused for the signed-in period, "sign out everywhere" is used, the password is changed or the second factor is enabled or disabled from another browser, or the recovery path is used.
- **FR-007**: The signed-in period MUST be a setting with a default of 30 days and an allowed range of 1 to 90 days. Each use of the application MUST extend the session by the full period. A changed period MUST apply to new sessions at once and to existing sessions from their next use.
- **FR-008**: System MUST let the person sign out of the current browser and, separately, sign out everywhere, which ends every session including the current one.
- **FR-009**: System MUST show the person a list of active sessions with, for each: when it started, when it was last used, a browser description, and the source address, with the current session marked.
- **FR-010**: Every screen and every piece of data MUST require a signed-in session, except: the account creation screen while no account exists, the sign-in screen, the recovery path, and a minimal "application is up" signal for the operator that reveals no personal data, no settings, and no health details.

*Throttling and logging*

- **FR-011**: System MUST record every sign-in attempt, every account creation attempt, every password change attempt, every second-factor code check, and every use of the recovery path with: time, kind of attempt (sign-in, account creation, password change, second-factor code, recovery), outcome (success, account created, creation refused, password changed, wrong password, wrong code, locked, refused, recovery), source address, browser description, and how long the attempt waited under FR-012, if it waited at all. For a use of the recovery path the source address is the host and the browser description is "recovery path". The record MUST NOT contain the password or code that was entered.
- **FR-012**: After 5 failed attempts within 10 minutes, System MUST make every further attempt wait before it is checked. With 5 failures in the last 10 minutes the next attempt waits 2 seconds; each further failure within those 10 minutes doubles the wait (4, 8, 16 seconds and so on); the wait never exceeds 5 minutes. While an attempt waits, the screen MUST say that attempts are being delayed and show the remaining wait. After the wait the attempt is checked normally: a correct password (and code, when required) signs the person in, and a wrong one is recorded as wrong password or wrong code. A delayed attempt produces one record, whose outcome is its final result and which notes how long it waited. An attempt submitted while another attempt is still waiting MUST be refused without being checked, MUST be recorded with the outcome "refused", and MUST NOT count as a failure. Failures older than 10 minutes MUST NOT count toward the threshold, so the wait shrinks and then disappears as old failures fall out of the 10-minute window.
- **FR-013**: After 20 consecutive failed attempts since the last success, System MUST lock sign-in for 60 minutes, MUST refuse attempts during the lock without checking the password, and MUST tell the person when the lock lifts. When the lock lifts, the consecutive failure count is unchanged, so one more failed attempt starts a new 60-minute lock; only a successful sign-in or a use of the recovery path resets the count. Sessions that were already signed in keep working during a lock.
- **FR-014**: System MUST show the signed-in person their sign-in activity for at least the last 30 days, newest first, and MUST keep attempt records for 90 days.
- **FR-015**: A wrong current password on the password change screen and a wrong second-factor code MUST count as failed attempts for throttling and locking.

*Settings*

- **FR-016**: System MUST store the settings below for the single account, show the stored values on the settings screen, and use the listed defaults until the person changes them.

| Setting | Allowed values | Default |
|---|---|---|
| Timezone | A recognised region name, for example Europe/Berlin | The installation default if one was configured, otherwise UTC, until the person confirms a timezone at first run |
| Week start day | Any of the seven days | Monday |
| Daily check-in | A time of day, plus weekdays only or every day | 17:30, weekdays only |
| Weekly review | A day of the week and a time of day | Friday 16:00 |
| Weekly plan proposal | A day of the week and a time of day | Sunday 18:00 |
| Monthly retrospective | An ordinal (first, second, third, fourth, or last), a day of the week, and a time of day | First Monday of the month, 17:30 |
| Quarterly review | A week of the calendar quarter (1 to 13), a day of the week, and a time of day. Week 1 begins on the first day of the quarter | Week 1, Monday, 17:30 |
| Assistant model | One of the models the application offers | The product default model |
| Effort per kind of task | One of low, medium, high, very high, set separately for roadmap drafting, re-planning, weekly planning, link suggestions, check-ins, and coaching | Roadmap drafting: very high. Re-planning: high. Weekly planning: high. Link suggestions: medium. Check-ins: medium. Coaching: medium |
| Monthly budget | 0.00 to 10,000.00 USD with at most two decimals | 40.00 USD |
| Autonomy level | "proposals only" or "auto-apply low-risk operations" | Proposals only |
| Signed-in period | 1 to 90 days | 30 days |

- **FR-017**: System MUST check every settings change against the allowed values, MUST refuse invalid values with a message naming the field and the allowed values, and MUST save nothing from a refused request.
- **FR-018**: A saved setting MUST take effect immediately for everything that reads it: the next schedule computation, the next assistant session, the next budget check, and the next displayed date or time. No restart is needed.
- **FR-019**: System MUST interpret check-in times, week boundaries, day boundaries, and month boundaries in the person's timezone. The settings screen MUST show the next occurrence of each configured check-in time in the person's timezone. After a timezone change or a change to a check-in time, the next occurrence MUST be recomputed in the new timezone and MUST never lie in the past. Running the schedules at those times belongs to the notifications and schedules feature, which reads the occurrences computed here.
- **FR-020**: System MUST record every settings change with the time, the setting, the previous value, and the new value.
- **FR-021**: System MUST require an explicit confirmation before saving the autonomy level "auto-apply low-risk operations". The confirmation MUST explain in plain words that only two operation types become automatic, plan item status and key result progress, and only when the person stated the fact in the same conversation; that these are applied as proposals approved on the person's behalf and can be undone; and that everything else still needs approval. "Proposals only" MUST be the value after installation.
- **FR-022**: A change of the assistant model MUST apply to assistant sessions started after the change. Sessions already running MUST keep the model they started with. This feature stores the setting and requires that a session record its model at the start (FR-024); the assistant features enforce that a running session does not switch.
- **FR-023**: The effort setting MUST exist separately for each of the six kinds of assistant task, and each kind MUST read only its own value.

*Assistant usage and budget*

- **FR-024**: System MUST record every request the assistant makes to the model as an assistant usage record with: time, the assistant session it belongs to, kind of task, model, effort, amount of input, amount of input stored for reuse by later requests, amount of input reused from earlier requests, amount of output (in the units the model's price list uses), cost in USD computed at the time of the request from the price list kept with the settings, duration, and how the request ended. An assistant session MUST NOT be treated as complete while any of its requests lacks a usage record. The price list is maintained with releases, is not edited by the person, and MUST be shown read-only on the usage page with the date it was last updated. System MUST NOT offer a model that has no price in the list. If a request is nevertheless made with a model that has no price, the record MUST be written with a cost of zero and flagged as unpriced, and a notice (FR-048) MUST be created.
- **FR-025**: System MUST total usage per day and per calendar month, overall, per kind of task, and per session, with days and months following the person's timezone.
- **FR-026**: The usage page MUST show, for the selected month (the current month by default): total spend in USD, the budget, the percentage used, the remaining amount, spend by kind of task, spend by day, the share of input reused from earlier requests overall and per kind of task, a list of the month's assistant sessions with kind, start time, and cost, the number of unpriced requests, the price list in use (read-only), and the budget state (normal, warning, or paused). When the budget is 0 USD the percentage used MUST be shown as "not applicable". Each session in the list MUST open to show its usage records with the fields listed in FR-024, newest first. Earlier months MUST remain selectable, with their figures unchanged.
- **FR-027**: The usage page MUST include every usage record written up to 60 seconds before the page loads.
- **FR-028**: When month-to-date spend first reaches 80 percent of the budget, System MUST set the budget state to "warning", show the warning on the usage page, and create a budget warning notice (FR-048) no later than the start of the next day in their timezone. The warning is issued once per crossing: if a budget change takes spend back below 80 percent and spend crosses it again, a new warning is issued.
- **FR-029**: When month-to-date spend reaches 100 percent of the budget, System MUST set the budget state to "paused". While paused, each background assistant task that becomes due MUST be skipped rather than run, the skip MUST be recorded with its reason, and a notice (FR-048) MUST be created when the pause begins and each time a task is skipped.
- **FR-030**: System MUST check the budget immediately before every background assistant task starts and before every interactive assistant session starts, using spend recorded up to that moment, so that the pause takes effect before the next task that follows the crossing.
- **FR-031**: While paused, starting an interactive assistant session MUST require the person's confirmation for that session. Spend from a confirmed session MUST still be recorded and counted.
- **FR-032**: The pause MUST lift automatically when the budget is raised above the month's spend or when a new month begins. Tasks skipped during the pause MUST NOT be run retroactively; the next scheduled occurrence runs normally.
- **FR-033**: Lowering the budget to or below the month's spend, including to 0 USD, MUST apply the warning or pause state immediately, before the next background task is considered.
- **FR-034**: At the start of each month in the person's timezone, month-to-date spend MUST restart at zero and the budget state MUST return to normal, without changing the previous month's figures.

*System health*

- **FR-035**: The usage page MUST show a health section with: whether the database is reachable, checked when the page loads, with the check time; whether the worker is alive, meaning it has reported within the last 5 minutes, with its last report time, "not reporting since" that time when it is older than 5 minutes, or "never reported"; the time of the last successful backup, the outcome and time of the most recent backup run with its reason on failure, or "no backup recorded yet"; and the application's current time in the person's timezone.
- **FR-036**: The background worker MUST report that it is alive at least once every minute while it is running, so that a stopped worker is detectable within six minutes.
- **FR-037**: System MUST keep a record of backup runs with start time, finish time, outcome, size, and reason on failure, for the health section to read. The backup process that writes these records is specified in the export, backup, and restore feature.
- **FR-038**: When the database is unreachable, every screen that can still render MUST show a plain message saying so instead of a blank page, an unexplained error, or stale data presented as current.
- **FR-039**: Health details MUST be visible only to the signed-in person. The operator signal in FR-010 MUST reveal only whether the application is up.

*Password change, recovery, and second factor*

- **FR-040**: Signed in, the person MUST be able to change the password by entering the current password and a new password that meets FR-003. On success, every other session MUST end and the change MUST be recorded.
- **FR-041**: System MUST provide a recovery path that lets the person who controls the host set a new password, show the current sign-in email address and set a new one, clear a lock and the failure count, and turn the second factor off, without being signed in. The path MUST NOT be reachable by anyone who only has network access to the application. Every use MUST be recorded in the sign-in attempt log with the host as source address and "recovery path" as browser description, and MUST end every session. The steps MUST be documented for the person.
- **FR-042**: The second factor MUST be optional and off after installation. Enabling it MUST require confirmation with a current one-time code before it takes effect. On enabling, System MUST show ten one-time recovery codes exactly once and tell the person to store them.
- **FR-043**: While the second factor is on, sign-in MUST require the password and then a current one-time code or an unused recovery code. Each recovery code MUST work once; after one is used the person MUST be told how many remain. The person MUST be able to generate a new set of recovery codes, which invalidates every earlier code.
- **FR-044**: Turning the second factor off MUST require the current password and a current one-time code or recovery code. Enabling or disabling the second factor MUST end every other session and MUST be recorded.

*Usability and privacy*

- **FR-045**: Every screen in this feature MUST be usable on a phone browser with one hand and with the keyboard alone, and MUST meet WCAG 2.1 AA colour contrast.
- **FR-046**: Dates, times, and numbers MUST be shown in the person's timezone and in the format of the browser's locale. The interface language is English.
- **FR-047**: This feature MUST NOT send personal data outside the host: password checks, timezone handling, health checks, and usage totals MUST work without calling any outside service.

*Notices*

- **FR-048**: System MUST keep a list of in-application notices with, for each: time, kind (budget warning, pause began, task skipped, model no longer offered, unpriced request), a plain message, and whether the person has dismissed it. Notices not yet dismissed MUST be shown at the top of the usage page and the settings screen until the person dismisses them. Dismissed notices MUST remain readable on the usage page for the month they belong to. The notifications and schedules feature later takes over their display and delivery.

*Installation and start-up*

- **FR-049**: The whole system MUST start from one command with one configuration file, and MUST show the account creation screen (or the sign-in screen once an account exists) within one minute of the host starting. The parts the health section reports on (the database, the background worker, and the record of backup runs) MUST be part of that start, so that this feature can be run and tested on its own.
- **FR-050**: The steps to start and upgrade the system, check backup status, and recover the account MUST be documented for the person who operates the host and kept with the application.

### Key Entities *(include if feature involves data)*

- **Account**: The single person's identity in Career OS. Holds the email address used to sign in, the display name, when it was created, the password (kept in a form that can be checked but never shown), whether the second factor is on, the unused recovery codes, the current failed-attempt count, and the time until which sign-in is locked, if any. Exactly one exists.
- **Session**: One signed-in browser belonging to the account. Holds when it started, when it was last used, when it expires, a browser description, and the source address. Ends by sign-out, expiry, "sign out everywhere", a password or second-factor change, or the recovery path.
- **Settings**: The account's preferences listed in FR-016: timezone, week start day, check-in times (daily, weekly review, weekly plan proposal, monthly retrospective, quarterly review), assistant model, effort per kind of task, monthly budget, autonomy level, and signed-in period. Each change is recorded with time, previous value, and new value. The price list used to cost assistant requests is kept alongside, maintained with releases, and read-only to the person.
- **Assistant usage record**: One entry per request the assistant makes to the model. Holds time, the assistant session it belongs to, kind of task, model, effort, input, input stored for reuse, reused input, output, cost in USD, whether it is flagged as unpriced, duration, and how the request ended. Written once and never changed.
- **Monthly usage summary**: The totals derived from usage records for one calendar month in the person's timezone: spend overall, per day, per kind of task, and per session, plus request counts. Compared with the budget to produce the budget state.
- **Budget state**: For the current month: the budget amount, spend to date, percentage used, the state (normal, warning, paused), when the warning was issued, when the pause began, when it lifted, and the list of tasks skipped during the pause with their reasons.
- **Health status**: The current picture of the system: whether the database is reachable and when that was checked, the worker's last report time and whether it counts as alive, the last successful backup time, the most recent backup run (start, finish, outcome, size, failure reason), and the application's current time.
- **Sign-in attempt log**: One entry per sign-in attempt, account creation attempt, password change attempt, second-factor code check, or recovery use. Holds time, kind of attempt, outcome, source address, browser description, and how long the attempt waited, if it waited. For a recovery use the source is the host and the description is "recovery path". Never holds a password or code. Kept for 90 days.
- **Notice**: One in-application message to the person, listed in FR-048: time, kind, message, and whether it was dismissed. Shown on the usage page and the settings screen until dismissed. The notifications and schedules feature later displays and delivers the same list.

Naming: "assistant session" is the glossary's Agent session, "background assistant task" is the glossary's Job, and the check-in times are the glossary's Schedules.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A person seeing the screens for the first time completes first-run setup (account creation plus timezone and week start day) in under 3 minutes and in one attempt.
- **SC-002**: With a correct password and no throttling in effect, sign-in takes under 2 seconds from submitting the form to seeing the requested page on a local network.
- **SC-003**: In 100 percent of test runs, closing and reopening the browser within the signed-in period needs no new sign-in, and after signing out 100 percent of protected pages ask for sign-in.
- **SC-004**: With repeated wrong passwords, the sixth attempt within 10 minutes is checked only after a 2-second wait and the seventh after a 4-second wait, the twenty-first consecutive attempt is refused, and 100 percent of attempts appear in the sign-in activity view within 5 seconds.
- **SC-005**: A saved settings change is visible on every screen that shows it within 1 second of saving, without any restart, and is used by the next schedule computation, budget check, and assistant session.
- **SC-006**: The usage page shows every recorded assistant request within 60 seconds of it being recorded, and the month total equals the sum of the records to the cent.
- **SC-007**: After spend reaches 100 percent of the budget, zero background assistant tasks start until the pause lifts; the first task due after the crossing is skipped and recorded.
- **SC-008**: After spend reaches 80 percent of the budget, the warning is on the usage page at its next load, and the budget warning notice is shown at the top of the usage page and the settings screen no later than the start of the next day in the person's timezone.
- **SC-009**: A stopped worker shows as not reporting within 6 minutes of its last report; a finished backup run appears in the health section within 1 minute of being recorded; an unreachable database is stated on the next page load.
- **SC-010**: Every page in this feature loads in under 1 second on a local network.
- **SC-011**: Every screen in this feature passes a phone-sized and keyboard-only walkthrough and a WCAG 2.1 AA contrast check.
- **SC-012**: Zero second accounts can be created once one exists, across every attempt in testing, and no password or one-time code ever appears in any record or log.
- **SC-013**: 100 percent of assistant requests have a usage record with a cost, verified by comparing requests with records in testing.
- **SC-014**: A fresh installation started with one command and one configuration file shows the account creation screen within 1 minute of the host starting, in 100 percent of test runs.

## Assumptions

- The person is the only user and also operates the host. There is no administrator role and no sharing in this feature.
- The sign-in name is the person's email address, and the account also has a display name. No email is sent; the address is only an identifier.
- Password rule, adopted from the security document: at least 12 characters. An optional check against a list of commonly used passwords held on the host may be added; it never sends a password anywhere.
- Throttling and locking, adopted from the security document: delays begin after 5 failures within 10 minutes and grow exponentially; sign-in locks after 20 consecutive failures. Chosen here and stated in FR-012 and FR-013: the first delay is 2 seconds, doubling with each further failure and capped at 5 minutes; a delayed attempt is held and then checked, producing one record; the lock lasts 60 minutes and leaves the failure count unchanged when it lifts; existing sessions keep working during a lock.
- Signed-in period, adopted from the security document: 30 days of no use by default, extended on each use. Chosen here: configurable from 1 to 90 days.
- Second factor, adopted from the security document: optional, off by default, with recovery codes shown once. Chosen here: ten codes per set.
- Recovery path: the architecture names recovery codes; a host-level reset is required so that the person who controls the host can always regain the only account. Because no email is ever sent, the sign-in email address can be forgotten like the password, so the recovery path can show it and set a new one. Its steps are documented for the person. The mechanism is left to the plan, which records this as an addition to ADR-0007 or as a new ADR.
- Phone access (Q7): the application is reached over a private network; no public exposure is assumed. If the person ever exposes it publicly, the second factor must be turned on first, as ADR-0007 requires.
- Check-in time defaults (Q4): daily check-in at 17:30 local time on weekdays; weekly review Friday 16:00; weekly plan proposal Sunday 18:00; monthly retrospective on the first Monday of the month; quarterly review in the first week of each calendar quarter (Q11: calendar quarters). Chosen here, because Q4 names no time of day for them: the monthly retrospective and the quarterly review default to 17:30, the same as the daily check-in; the quarterly review's week 1 begins on the first day of the calendar quarter, and the quarterly review defaults to the Monday of week 1, because Q4 names no day. These settings are stored here and consumed by the notifications and schedules feature.
- Growth work happens on weekdays (Q2), which is why daily check-ins default to weekdays only. The weekly hours figure itself belongs to the profile feature and is not a setting here.
- Starting budget (Q10): 40 USD per month. Chosen here: the budget may be set from 0 to 10,000 USD with at most two decimals; 0 pauses all background work; tasks skipped during a pause are not re-run.
- Language and locale (Q15): English only; dates and numbers follow the browser's locale; times follow the timezone setting.
- Week start day default: Monday. Timezone before the person sets one: the installation default if configured, otherwise UTC; the first-run screen pre-fills the browser's timezone where available.
- Assistant model and effort, adopted from the agent architecture and ADR-0006: one model for all kinds of task, chosen from a list the application offers and updated with releases; effort is set per kind of task on a four-level scale with defaults of very high for roadmap drafting, high for re-planning and weekly planning, and medium for link suggestions, check-ins, and coaching. The price list used to cost requests is kept with the settings, updated with releases, and shown read-only on the usage page. Chosen here: a request made with a model that has no price is recorded with a cost of zero, flagged as unpriced, and announced with a notice, so that the gap is visible rather than silently guessed.
- Autonomy level, adopted from the constitution and ADR-0005: "proposals only" by default; "auto-apply low-risk operations" is the only other value, and it covers only two operation types, plan item status and key result progress, only when the person stated the fact in the same conversation, applied as auto-approved proposals that can be undone. The confirmation text in FR-021 says exactly that. This feature stores the setting; the proposals feature enforces it.
- Budget rules, adopted from the product requirements, ADR-0006, and the constitution: warning at 80 percent, delivered no later than the start of the next day; pause of background assistant work at 100 percent with interactive sessions gated by a confirmation; the budget is checked before every background task and every interactive session. Months are calendar months in the person's timezone.
- Health semantics, adopted from the deployment document: the worker reports every minute and counts as stale after 5 minutes; the last backup time comes from recorded backup runs. The backup process itself belongs to the export, backup, and restore feature. These health details are shown only to the signed-in person; the operator's automated check is a separate minimal signal that reveals only that the application is up (FR-010, FR-039). This is deliberately stricter than the deployment document, which lets the operator signal report worker staleness; the plan records the reconciliation.
- Repository bootstrap: as the first feature with no predecessor, this one carries the start of the whole system (handoff guide, section 5): the workspace layout, the one-command start with one configuration file, and the automated checks. FR-049 states the requirement in plain terms; the mechanisms belong to the plan.
- Sign-in attempt records are kept for 90 days and the activity view shows at least the last 30 days.
- Design note for feature 011: the notice list in FR-048 is expected to be reused as is by the notifications feature.
- Until the first assistant feature exists, the budget pause and the usage recording rules are verified with a stand-in background task that only asks the budget gate and writes a usage record.
- The person's browser keeps the signed-in state across restarts; clearing browser data signs that browser out, which is expected.
- Source requirements: docs FR-001, FR-002, FR-003, FR-004; stories US-001, US-002, US-003; NFR-001, NFR-002, NFR-003, NFR-005, NFR-007, NFR-008, NFR-009, NFR-010, NFR-013; constitution principles I, II, III, VII, IX; ADR-0005, ADR-0006, ADR-0007. FR-002's schedules are covered here except the nightly re-plan check time, which feature 011 configures.

## Out of Scope

This feature deliberately leaves the following to other features:

- Profile and self-assessment (feature 002), including weekly hours available for growth work.
- Roadmap, milestones, and activities (feature 003).
- The proposals inbox and enforcement of the autonomy level on assistant output (feature 004).
- The notifications feature (feature 011): the notification centre, delivery channels, and reminder scheduling. This feature keeps the notices listed in FR-048 (budget warning, pause began, task skipped, model no longer offered, unpriced request) visible at the top of the usage page and the settings screen until dismissed; feature 011 later takes over their display and delivery.
- Running the schedules that use the check-in time settings (feature 011). This feature stores the times and shows their next occurrence; it fires nothing.
- The nightly re-plan check time and any schedule not listed in FR-016 are configured in the notifications and schedules feature (feature 011).
- The assistant itself and its sessions (features 005, 008, 012), including the per-session caps on iterations and tokens (NFR-013) and the warning before an interactive session exceeds its cap (constitution VII). This feature covers only the monthly budget, defines what each request must record, and states how the budget gates assistant work.
- Export, backups, and restore (feature 014); this feature only shows the recorded backup runs.
- Sharing with a mentor or manager, additional accounts, roles, external identity providers, and passkeys.
