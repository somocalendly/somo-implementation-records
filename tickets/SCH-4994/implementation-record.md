# SCH-4994 — Enable staging canary transition for booking

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4994
- Type: Task
- Story points: not recorded
- Repo: `calendly/booking`
- Branch: canary-booking-staging-transition
- Pull request: https://github.com/calendly/booking/pull/496

## Timeline

- Started: 2026-09-15
- Pull request opened: 2026-09-15
- Released: 2026-10-22

## What changed

Staging still shipped `booking-main` as a plain Deployment, so merges rolled pods in place with no canary handoff. Staging now runs the first canary step: a Rollout beside the existing Deployment until the Rollout is healthy, then the Deployment scales to zero.

## Work

- Turned on staging canary transition for `booking-main`, with Slack notifications to the scheduling alerts channel (`values/staging.yaml`).
