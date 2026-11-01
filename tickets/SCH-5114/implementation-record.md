# SCH-5114 — Enable production canary transition for booking

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5114
- Type: Sub-task
- Story points: not recorded
- Repo: `calendly/booking`
- Branch: canary-booking-production-transition
- Pull request: https://github.com/calendly/booking/pull/498

## Timeline

- Started: 2026-09-15
- Pull request opened: 2026-09-15
- Released: 2026-10-27

## What changed

Production still shipped `booking-main` as a plain Deployment while staging had already moved to a Rollout. Production now runs the canary transition step: a Rollout beside the Deployment until the Rollout is healthy, then the Deployment scales to zero.

## Work

- Turned on production canary transition for `booking-main`, with Slack notifications to the scheduling alerts channel (`values/production.yaml`).
