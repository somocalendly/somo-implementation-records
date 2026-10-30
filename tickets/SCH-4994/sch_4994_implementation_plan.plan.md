---
name: SCH-4994 staging canary transition
overview: SCH-4994 deploys as four sequenced subtasks. This plan is only SCH-5112, the staging transition. SCH-5113, SCH-5114, and SCH-5115 each wait for the previous deploy to be healthy.
branch: canary-booking-staging-transition
issueType: Task
pullRequest: https://github.com/calendly/booking/pull/496
started: 2026-09-15
pullRequestOpened: 2026-09-15
released: 2026-10-22
delivery:
  - repo: calendly/booking
    branch: canary-booking-staging-transition
    pullRequest: https://github.com/calendly/booking/pull/496
todos:
  - id: staging-transition
    content: Add canary.enabled, transition, and notifications under applications.main in values/staging.yaml
    status: pending
isProject: false
---

# Enable staging canary transition for booking

Implementation plan for [SCH-4994](https://calendlyapp.atlassian.net/browse/SCH-4994) (ET Scheduling · Task · P1 · Backlog). Parent epic is [SCH-3790](https://calendlyapp.atlassian.net/browse/SCH-3790), "Misc non-KTLO: Scheduling." It was [SCH-3652](https://calendlyapp.atlassian.net/browse/SCH-3652), "2026 QoL KTLO: Scheduling," until the parent was moved on 2026-10-21. Infra's round-two tracker is [IN-8760](https://calendlyapp.atlassian.net/browse/IN-8760), status Done since 2026-10-20. Its closing comment leaves booking on this ticket.

This plan implements [SCH-5112](https://calendlyapp.atlassian.net/browse/SCH-5112) only. It does not finish [SCH-4994](https://calendlyapp.atlassian.net/browse/SCH-4994).

## Sequence

SCH-4994 is one migration and four deploys. Each subtask is its own pull request and its own merge to `main`. The next subtask starts only after the previous deploy is healthy. Do not put two of these in one pull request.

Transition and native are separate applies. Transition runs a Rollout beside the existing Deployment until the Rollout is healthy and the Deployment scales to zero. Native then removes the Deployment from desired state. One apply would skip that handoff. Production waits until the staging pair is done. Production runs about 35 pods at 3.25 CPU and 7.92G each, so the transition overlap is a capacity event.

| Order | Subtask | Start when | What ships | Stale pull request |
| --- | --- | --- | --- | --- |
| 1 | [SCH-5112](https://calendlyapp.atlassian.net/browse/SCH-5112) | Now. This plan. | Staging `transition: true` | [#496](https://github.com/calendly/booking/pull/496), Leigh Jia's pull request. Use it. Update it onto current `main`. Do not open a second pull request. |
| 2 | [SCH-5113](https://calendlyapp.atlassian.net/browse/SCH-5113) | SCH-5112 is on `main` and staging stayed healthy. This step. | Remove `transition` in staging, then prune the Deployment in Argo CD | [#497](https://github.com/calendly/booking/pull/497). On 2026-10-22 `main` was merged in. The only diff is deleting `transition: true`. |
| 3 | [SCH-5114](https://calendlyapp.atlassian.net/browse/SCH-5114) | SCH-5113 is pruned and a staging booking still succeeds. Re-check production replica count first. | Production `transition: true` in `values/production.yaml` only | [#498](https://github.com/calendly/booking/pull/498). Do not merge it. Its diff also rewrites `values/staging.yaml`. |
| 4 | [SCH-5115](https://calendlyapp.atlassian.net/browse/SCH-5115) | SCH-5114 Rollout is Healthy and the production Deployment is at zero. Do this promptly. | Remove `transition` in production, then prune | [#499](https://github.com/calendly/booking/pull/499). Do not merge it. Its diff also touches `values/staging.yaml`. |

[#496](https://github.com/calendly/booking/pull/496) is Leigh Jia's staging transition pull request. On 2026-10-21 it was updated onto current `main`. Use that pull request. [#497](https://github.com/calendly/booking/pull/497)–[#499](https://github.com/calendly/booking/pull/499) are still the later stages, open and unreviewed, and still based on the 2026-09-08 commit `c68cda38`. Update each one onto `main` when its turn comes. Do not open a second pull request for the same stage.

[booking#500](https://github.com/calendly/booking/issues/500) is not a fifth subtask. Datadog analysis and automated rollback start after SCH-5115.

Repo: `calendly/booking`. Default branch is `main`. Checkout for this run:

```bash
git clone git@github.com:calendly/booking.git /Users/somo.banerjee/Documents/projects/SCH-4994/booking
```

Branch: `canary-booking-staging-transition`

Pull request: https://github.com/calendly/booking/pull/496

Started: 2026-09-15. Pull request opened: 2026-09-15. Released: 2026-10-22.

That clone is on `main` at `ec95a3d2` (`SCH-5096`). The chart pin for that commit is `calendly-service` 3.3.0. Today's staging deploy and production promote both passed `calendly-service-chart-version: 3.3.0` (workflow run 39669825998). `charts` `calendly-service` `Chart.yaml` on the default branch is also 3.3.0. That is past the 1.95.0 minimum in the [Canary Migration Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5618663471/Canary+Migration+Guide). [booking#500](https://github.com/calendly/booking/issues/500) recorded 2.9.0 on 2026-09-15. That figure is stale.

Chart 3.3.0 still documents the same handoff this plan uses. In `charts/calendly-service/values.yaml`, `canary.transition` is "Set true for zero-downtime handoff from Deployment to Rollout," `scaleDown` still defaults to `onsuccess`, and the default steps are still `setWeight: 20`, pause 60s, then `analysis`. The [Canary Reference Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5619351578/Canary+Reference+Guide) does not mention 3.x. The chart values do.

Booking deploys with GitHub Actions (`.github/workflows/deploy.yml`) and promotion-bot. Helm values live in `values/`, not `chart/values-*.yaml` or `k8s-values-*.yaml`. The only workload is `applications.main` (`booking-main`) in namespace `booking`.

## Goal

`main` still ships `booking-main` as a normal Kubernetes Deployment. `values/staging.yaml` and `values/production.yaml` have no `canary` key. The live production Deployment is labeled `calendly.com/canary-stage:disabled`. A merge to `main` replaces pods with a rolling update and then promotes to production in the same workflow.

Canary, for this step, means staging creates an Argo Rollout next to that Deployment (`transition: true`, `scaleDown: onsuccess`). When the Rollout is healthy, the controller scales the Deployment to zero. Later deploys in staging use the chart default steps: 20% pod weight, pause 60 seconds, then promote. There is no analysis block, so nothing rolls back on a metric.

Infra already opened that change as [booking#496](https://github.com/calendly/booking/pull/496), authored by Leigh Jia. This run uses that pull request. On 2026-10-21 `main` was merged into `canary-booking-staging-transition`, so the pull request is current and the only diff from `main` is the canary block. Do not open a second pull request.

## Out of scope

- [SCH-5113](https://calendlyapp.atlassian.net/browse/SCH-5113), [SCH-5114](https://calendlyapp.atlassian.net/browse/SCH-5114), and [SCH-5115](https://calendlyapp.atlassian.net/browse/SCH-5115). See Sequence. Do not leave staging in transition mode, and do not start production in this pull request.
- Datadog analysis, `dryRun`, and automated rollback. That is [booking#500](https://github.com/calendly/booking/issues/500), after SCH-5115.
- `values/production.yaml`, `values/base.yaml`, and `values/devspaces.yaml`. DevSpaces has no Argo Rollouts controller.
- Cloud SQL auth-proxy selectors. `infra-values.yaml` has no `cloudsql` block. Production uses Aiven.
- A second workload. There is no worker.
- Probes, HTTPRoutes, autoscaling queries, replica min/max, resources, configMaps, and application code.
- Opening a second pull request for a stage that already has one. [#497](https://github.com/calendly/booking/pull/497)–[#499](https://github.com/calendly/booking/pull/499) still need `main` merged in when their turn comes.

## Code changes

`booking/values/staging.yaml`. Under `applications.main`, before the autoscaling block, add:

```yaml
    canary:
      enabled: true
      transition: true
      notifications:
        enabled: true
        slack:
          channel: "#alerts-scheduling-and-availability"
```

That matches #496. The channel is the one infra put on the prepared stack and the one existing booking monitors already use. Confirm it at review. The Argo Rollouts bot has to be in that channel or the notifications do not arrive. Default triggers stay: rollout completed, rollout aborted, analysis failed, analysis error.

Leave the rest of the file as it is, including hosts (`staging.calendly.com`, `calendly.jasoncarter.xyz`), paths (`/api/booking/`, `/assets/booking/`, `/`), 60s timeouts, min/max replicas 2/5, and the configMap. Probes and the puma autoscaling queries stay in `values/base.yaml`.

Do not set `strategy.steps`, `analysis`, or `skipCanary`.

## Tests

No spec reads `values/`. Do not add one. RSpec, Jest, and Brakeman do not cover this file. A green CI run is not evidence.

After the edit, the only diff should be the block above in `values/staging.yaml`.

## Dev test

This does not change the process that runs on a laptop. Signadot and a local booking page do not load the staging Rollout.

The check is after this pull request merges to `main`. `.github/workflows/deploy.yml` deploys that merge to staging.

In Argo CD, open the `booking` application:

- A Rollout for `booking-main` exists and is Healthy.
- The old `booking-main` Deployment has scaled to zero.
- The application is Synced and Healthy. It does not self-heal.
- Staging Rollouts UI: `https://argo-rollouts.staging.calendly-internal.com/rollouts/booking`

Then open a public scheduling page on `staging.calendly.com` and confirm it renders availability. That page load is the proof this step did not take booking down. It is not proof of a later canary step, because this merge is the transition itself.

Before requesting review, confirm the live Deployment label is `helm.sh/chart: calendly-service-3.3.0`.

## Manual check

Staging only. Flag: none. Account: any staging host with a public event type.

- [ ] Rollout `booking-main` is Healthy and the Deployment is at zero replicas.
- [ ] A public scheduling page on `staging.calendly.com` shows real availability.
- [ ] Book a slot. The event appears once for the host, once on the connected calendar, and the invitee gets one confirmation.
- [ ] Reschedule that booking, then cancel it. Each action updates the same event.
- [ ] A payment-required event type reaches its payment step.
- [ ] Page assets load from `/assets/booking/` with a clean browser console.
- [ ] Production `values/production.yaml` is unchanged, and production booking still serves on the current Deployment.

Rollback is reverting the pull request. Do not delete the Rollout or Deployment by hand.

## Sources

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4994 — description is only the Slack thread https://calendly.slack.com/archives/C01GEFCMK43/p1789581182423549. That thread was not readable in this session. [IN-8760](https://calendlyapp.atlassian.net/browse/IN-8760) is Done. Its description still says booking is "waiting on team." The 2026-10-20 closing comment says the booking ticket is filed as SCH-4994 and "not started yet," and that remaining work is team-owned. Those two lines describe the same handoff.
- [Canary Migration Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5618663471/Canary+Migration+Guide) — step 1 is `enabled: true` plus `transition: true` in the staging values file. Health checks are a later step. Both versions run at once during the handoff.
- [Canary Reference Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5619351578/Canary+Reference+Guide) — default steps are `setWeight: 20`, pause 60s, then `analysis` (skipped when analysis is off).
- Prepared stack, still open: [booking#496](https://github.com/calendly/booking/pull/496), [#497](https://github.com/calendly/booking/pull/497), [#498](https://github.com/calendly/booking/pull/498), [#499](https://github.com/calendly/booking/pull/499). Health-check research: [booking#500](https://github.com/calendly/booking/issues/500).
- Closed earlier attempts, not reused: [booking#17](https://github.com/calendly/booking/pull/17), [booking#18](https://github.com/calendly/booking/pull/18).
