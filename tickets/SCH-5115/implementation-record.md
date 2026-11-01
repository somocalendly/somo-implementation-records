# SCH-5115 — Move production booking to a native Rollout

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5115
- Type: Sub-task
- Story points: not recorded
- Repo: `calendly/booking`
- Branch: canary-booking-production-native
- Pull request: https://github.com/calendly/booking/pull/499

## Timeline

- Started: 2026-09-15
- Pull request opened: 2026-09-15
- Released: 2026-10-29

## What changed

After the production transition merge, `transition: true` still kept the old Deployment in desired state. Production now runs `booking-main` as a native Rollout only, which completes the SCH-4994 migration in production.

## Work

- Removed production `transition: true` so the Rollout is the sole controller for `booking-main`, with canary and Slack notifications unchanged (`values/production.yaml`).
