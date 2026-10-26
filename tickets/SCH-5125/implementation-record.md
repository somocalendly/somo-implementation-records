# SCH-5125 — Fork staging sandboxes from the booking Rollout

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5125
- Type: Sub-task
- Story points: not recorded
- Repo: `calendly/booking`
- Branch: SCH-5125-fork-sandbox-from-rollout
- Pull request: https://github.com/calendly/booking/pull/543

## Timeline

- Started: 2026-10-25
- Pull request opened: 2026-10-25
- Released: 2026-10-26

## What changed

After staging booking moved to a Rollout-only `booking-main`, Signadot sandbox publish still tried to fork a Deployment with that name and failed. The staging overlay workflow now passes `fork-kind: Rollout` so labeled pull requests can publish sandboxes again.

## Work

- Set Signadot deploy-sandbox to fork the staging Rollout instead of a missing Deployment when the overlay sandbox label is on the pull request (`.github/workflows/sandbox-deploy.yml`).
