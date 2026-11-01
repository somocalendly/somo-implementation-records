---
name: SCH-5114 production canary transition
overview: Turn on Argo Rollouts transition mode for booking-main in production by updating booking#498 onto current main. Staging stays as it is. The native production step is SCH-5115.
branch: canary-booking-production-transition
issueType: Sub-task
pullRequest: https://github.com/calendly/booking/pull/498
started: 2026-09-15
pullRequestOpened: 2026-09-15
released: 2026-10-27
delivery:
  - repo: calendly/booking
    branch: canary-booking-production-transition
    pullRequest: https://github.com/calendly/booking/pull/498
todos:
  - id: update-498
    content: Merge current main into canary-booking-production-transition so the pull request diff stays the production canary block only
    status: pending
  - id: retitle-498
    content: Set the booking#498 title to include SCH-5114 and replace the September capacity numbers in the body
    status: pending
isProject: false
---

# Enable production canary transition for booking

Implementation plan for [SCH-5114](https://calendlyapp.atlassian.net/browse/SCH-5114) (ET Scheduling · Sub-task · P1 · In Dev). Parent is [SCH-4994](https://calendlyapp.atlassian.net/browse/SCH-4994). This file is the design document and the implementation plan for this subtask only.

Repo: `calendly/booking`. Default branch is `main`, currently `91ccf951` (merge of [#497](https://github.com/calendly/booking/pull/497)).

Branch: `canary-booking-production-transition`

Pull request: https://github.com/calendly/booking/pull/498

Started: 2026-09-15. Pull request opened: 2026-09-15. Released: 2026-10-27.

Checkout for this run:

```bash
git clone git@github.com:calendly/booking.git /Users/somo.banerjee/Documents/projects/SCH-4994/booking
```

That clone is clean and still checked out on `canary-booking-staging-native`. This plan does not edit it. After approval, work continues on the existing branch `canary-booking-production-transition`.

## Goal

Production `booking-main` is still a normal Deployment. `values/production.yaml` on `main` has no `canary` key. A merge to `main` deploys staging, then promotes production in `.github/workflows/deploy.yml`. Staging is already a native Rollout ([SCH-5113](https://calendlyapp.atlassian.net/browse/SCH-5113), [#497](https://github.com/calendly/booking/pull/497)).

This step creates a production Rollout beside that Deployment. When the Rollout is healthy, the controller scales the Deployment to zero. Both serve the same Service during the overlap, so requested capacity about doubles until scale-down.

## Design

Use [booking#498](https://github.com/calendly/booking/pull/498). Leigh Jia opened it. Its base is already `main`. It is 79 commits behind and 1 ahead. That one commit, `063d8b30`, adds the canary block to `values/production.yaml` and nothing else. `git diff origin/main...origin/canary-booking-production-transition` is those 7 lines. Merging current `main` into the branch is clean (`git merge-tree` exit 0).

Do not open a second pull request. The Jira sentence that says to open a new branch was written when this pull request still rewrote `values/staging.yaml`. That is no longer the GitHub diff. After approval, correct that Jira sentence to say use #498.

Rejected: stacking [#499](https://github.com/calendly/booking/pull/499) onto this pull request, and merging this before the branch contains current `main`. A merge of the branch as it sits would still ship only the production block, but GitHub reports the pull request behind, and the body still describes the September fleet.

Current production requests, from `values/production.yaml` on `main`:

| | |
| --- | --- |
| minReplicas | 20 |
| maxReplicas | 250 |
| CPU request | 2.0 |
| Memory request | 7.92G |
| Memory limit | 22.35G |

[#540](https://github.com/calendly/booking/pull/540) lowered the CPU request from 3.25 to 2.0. [#541](https://github.com/calendly/booking/pull/541) lowered `minReplicas` from 35 to 20. The #498 body still says 35 pods and 3.25 CPU. At the floor, overlap is about +20 pods, +40 CPU, and +158 GB of memory requests. The HPA can be above 20, and `setWeight` is not involved in this handoff. The overlap is one full extra copy of whatever the HPA wants, until the Deployment scales to zero. Re-check ready replicas in the production Argo app just before merge. This plan cannot read that count.

Hosts stay `calendly.com`, `calendly.vanguard.com`, and `calendly.alliancebernstein.com`. No Cloud SQL change. Production uses Aiven. No worker. No `analysis`, `dryRun`, `strategy.steps`, or `skipCanary`.

Chart behavior is unchanged from the staging transition: `canary.transition` is the zero-downtime handoff, `scaleDown` defaults to `onsuccess`. The [Canary Migration Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5618663471/Canary+Migration+Guide) step 5 says repeat the staging transition in the production values file, as its own change, after staging steps 1 and 2 are verified.

## Out of scope

- [SCH-5115](https://calendlyapp.atlassian.net/browse/SCH-5115) / [#499](https://github.com/calendly/booking/pull/499). Do not remove `transition: true` in this pull request, and do not prune the production Deployment. Prune is the next subtask, after this Rollout is Healthy and the Deployment is at zero.
- Datadog analysis and automated rollback. That is [booking#500](https://github.com/calendly/booking/issues/500).
- `values/staging.yaml`, `values/base.yaml`, `values/devspaces.yaml`, probes, routes, replica bounds, resource requests, and application code.
- Re-running the failed staging job for [#497](https://github.com/calendly/booking/pull/497) (Actions run 40126400997).
- Promoting commit `91ccf951` to production by itself. That commit does not change production values.

## Code changes

[Canary Migration Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5618663471/Canary+Migration+Guide) step 5, applying step 1 to the production values file. The file this repo actually deploys is `values/production.yaml`, not `chart/values-production.yaml`. The guide's path table is the generic layout. Booking's workflow reads `values/`.

The block is already on `canary-booking-production-transition`, inserted under `applications.main` before the autoscaling block:

```yaml
    canary:
      enabled: true
      transition: true
      notifications:
        enabled: true
        slack:
          channel: "#alerts-scheduling-and-availability"
```

Keep that channel. It is the one staging already uses. `package.yml` lists `#squad-scheduling-platform` as the squad channel. That is not the notification channel. The Argo Rollouts bot has to be in `#alerts-scheduling-and-availability`. Staging already required that.

After approval:

1. Check out `canary-booking-production-transition` in the ticket clone.
2. Merge `origin/main` into it. No force-push. No new canary commit unless the merge does not apply the block.
3. Confirm `git diff origin/main` is only that block in `values/production.yaml`, and that `minReplicas` stays 20 and the CPU request stays 2.0.
4. Push the merge.
5. Set the title to `[SCH-5114] [production] Canary 3/4: enable transition mode for booking` so Jira links the pull request.
6. Replace the body's September capacity paragraph with the table above, and say the live replica count is re-checked at merge time. Leave Leigh's description of the handoff in place.

Do not retarget the pull request. Do not stack #499.

## Testing

No spec reads `values/`. Do not add one. RSpec, Jest, and Brakeman do not cover this file. Booking has a frontend, and this change does not alter a screen, so there is no Storybook story. `package.yml` and a repo search found no `CODEOWNERS` file.

Local, before commit and push: `git diff origin/main` is the block above and nothing else. Merging `main` is clean.

Shared environment, before review: there is no Signadot or staging apply for this file. Staging is already a native Rollout, and this pull request must not change it. The pre-review proof is the production-only diff plus a fresh read of production ready replicas and the HPA in Argo. Also re-read Datadog monitors `16732224` (Booking Creation Reliability SLO) and `16732837` (Booking Update Latency SLO). The #498 body says both were alerting on 2026-09-15. That date is not the current reading.

## Post-production

The event is the merge to `main`. `deploy-staging` runs first. Staging values do not change, so that job should go Healthy without a prune. `deploy-production` then waits on the GitHub `production` environment. An Engineers reviewer has to approve that promote. This merge is the production change, unlike #497.

After the production promote, in Argo CD and at https://argo-rollouts.production.calendly-internal.com/rollouts/booking:

- Rollout `booking-main` is Healthy.
- The old Deployment has scaled to zero. It is still in the app until SCH-5115 prunes it.
- The `booking` application is Synced and Healthy.
- No pods are Pending from the extra copy.
- Pod count is back in its normal range.

Then a public page on `calendly.com` shows availability. Book one event, reschedule it, and cancel it, each once. A payment-required event type reaches its payment step. Spot-check `calendly.vanguard.com` and `calendly.alliancebernstein.com`. Assets load from `/assets/booking/`.

Start SCH-5115 promptly once that Rollout is Healthy and the Deployment is at zero. Do not prune in this step.

## Rollback

Revert the pull request. That removes the canary block and returns desired state to a Deployment. Do not delete the Rollout or the Deployment by hand. A revert does not undo a booking, a payment, or a calendar write that already happened. The guide's `canary.revert: true` then `enabled: false` path is the later undo, after a native Rollout. This step is still transition, so the revert of the pull request is the undo.

## Sources

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5114
- Parent: https://calendlyapp.atlassian.net/browse/SCH-4994
- [Canary Migration Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5618663471/Canary+Migration+Guide) — step 1 is `enabled: true` plus `transition: true`; step 5 repeats that in the production values file after staging is verified. Health checks are steps 3 and 4 and stay out of this pull request. The guide's `chart/values-*.yaml` paths are the generic GitHub Actions layout. This repo uses `values/`.
- [Canary Reference Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5619351578/Canary+Reference+Guide) — default steps after this handoff are `setWeight: 20`, pause 60s, then analysis. Analysis is skipped when it is not configured.
- Existing pull request: https://github.com/calendly/booking/pull/498
- Prior stages: https://github.com/calendly/booking/pull/496 and https://github.com/calendly/booking/pull/497
- Next stage, not this plan: https://github.com/calendly/booking/pull/499 and https://github.com/calendly/booking/issues/500

## Review

No outside team reviews this diff. The repo has no `CODEOWNERS` file. `package.yml` stewards are `@calendly/scheduling-platform`, the squad that owns the ticket. Production deploy protection is separate: the `production` environment in `.github/workflows/deploy.yml` requires an Engineers reviewer at promote time. That approval is not a review of this YAML. Request the same squad reviewer used on #496 and #497. No Slack post is required for the diff. Notifications after deploy go to `#alerts-scheduling-and-availability`.
