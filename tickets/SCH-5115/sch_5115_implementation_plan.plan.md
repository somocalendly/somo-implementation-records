---
name: SCH-5115 production native rollout
overview: Remove production transition mode by updating booking#499 onto current main, then prune the zero-replica Deployment. This finishes the SCH-4994 baseline. Analysis stays in booking#500.
branch: canary-booking-production-native
issueType: Sub-task
pullRequest: https://github.com/calendly/booking/pull/499
started: 2026-09-15
pullRequestOpened: 2026-09-15
released: 2026-10-29
delivery:
  - repo: calendly/booking
    branch: canary-booking-production-native
    pullRequest: https://github.com/calendly/booking/pull/499
todos:
  - id: update-499
    content: Merge current main into canary-booking-production-native so the pull request diff stays the deletion of transition true
    status: pending
  - id: retitle-499
    content: Set the booking#499 title to include SCH-5115 and correct the two stale sentences in the body
    status: pending
isProject: false
---

# Move production booking to a native Rollout

Implementation plan for [SCH-5115](https://calendlyapp.atlassian.net/browse/SCH-5115) (ET Scheduling · Sub-task · P1 · In Dev). Parent is [SCH-4994](https://calendlyapp.atlassian.net/browse/SCH-4994). This file is the design document and the implementation plan for this subtask only.

Repo: `calendly/booking`. Default branch is `main`, currently `06aa024f` (merge of [#498](https://github.com/calendly/booking/pull/498)).

Branch: `canary-booking-production-native`

Pull request: https://github.com/calendly/booking/pull/499

Started: 2026-09-15. Pull request opened: 2026-09-15. Released: 2026-10-29.

Checkout for this run:

```bash
git clone git@github.com:calendly/booking.git /Users/somo.banerjee/Documents/projects/SCH-4994/booking
```

That clone is clean and still checked out on `canary-booking-production-transition`. This plan does not edit it. After approval, work continues on the existing branch `canary-booking-production-native`.

## Goal

Production `booking-main` is a Healthy Canary Rollout at weight 100, revision 1, with 20 ready pods on ReplicaSet `booking-main-5ffdbd95c7`. The old Deployment still exists. Its live manifest has `spec.replicas: 0`, label `calendly.com/canary-stage: transition`, and commit `06aa024f`. Argo CD reports that Deployment Healthy and Synced. Auto-prune is off, so it stays until someone prunes it.

`values/production.yaml` on `main` still has `transition: true`. Leaving that flag keeps the Deployment in desired state.

## Design

Use [booking#499](https://github.com/calendly/booking/pull/499). Leigh Jia opened it. Its base is already `main` at `06aa024f`. GitHub shows one deletion. The branch is 100 commits behind and 1 ahead. That one commit, `65732964`, deletes `transition: true`. Merging current `main` into the branch is clean. The merged file keeps `canary.enabled: true`, the Slack channel, `minReplicas: 20`, and CPU request `2.0`.

Do not open a second pull request. The Jira sentence that says to branch from `main` was written when this pull request still touched `values/staging.yaml`. That is no longer the GitHub diff. After approval, correct that Jira sentence to say use #499.

This step does not add pods. The Rollout already serves traffic. Removing `transition` stops the chart from generating the Deployment. The object remains in the cluster until the production app is synced with Prune. That is the same cleanup staging needed after [#497](https://github.com/calendly/booking/pull/497).

Rejected: deleting the Deployment from its Argo CD detail panel. The guide's cleanup is the application Sync with Prune checked. The detail panel also has a Delete button. Do not use it.

## Out of scope

- Datadog analysis, `dryRun`, and automated rollback. That is [booking#500](https://github.com/calendly/booking/issues/500). Do not paste its starter YAML here.
- `values/staging.yaml`, replica bounds, resource requests, routes, probes, and application code.
- A second pull request.
- Clicking Delete on the Deployment.

## Code changes

[Canary Migration Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5618663471/Canary+Migration+Guide) step 5, applying step 2 to the production values file. Booking deploys `values/production.yaml`, not `chart/values-production.yaml`.

The deletion is already on `canary-booking-production-native`. After the merge, `applications.main.canary` is:

```yaml
    canary:
      enabled: true
      notifications:
        enabled: true
        slack:
          channel: "#alerts-scheduling-and-availability"
```

After approval:

1. Check out `canary-booking-production-native` in the ticket clone.
2. Merge `origin/main` into it. No force-push.
3. Confirm `git diff origin/main` is only the deleted `transition: true` line, and that `minReplicas` stays 20 and the CPU request stays 2.0.
4. Push the merge.
5. Set the title to `[SCH-5115] [production] Canary 4/4: move booking to a native Rollout`.
6. In Leigh's description, change only the stale facts. The sentence that says replica counts stay at 35 to 250 becomes 20 to 250, and says [#541](https://github.com/calendly/booking/pull/541) lowered the floor after the description was written. This pull request does not change the autoscaler. The sentence that says the Part 3 end-to-end booking check passed becomes: a public page on `calendly.com` showed selectable days. A book, reschedule, and cancel were not run. Leave the rest of the description as Leigh wrote it.

## Testing

No spec reads `values/`. Do not add one. This change does not alter a screen in the repo, so there is no Storybook story. The repo has no `CODEOWNERS` file. `package.yml` stewards are `@calendly/scheduling-platform`.

Local, before commit and push: `git diff origin/main` is that one deleted line.

Shared environment, before review: there is no staging apply for this file. Staging is already a native Rollout. The pre-review proof is the one-line production diff. Production proof is after merge.

## Post-production

The event is the merge to `main`. [Deploy run for #498](https://github.com/calendly/booking/actions/runs/40604951347) showed the shape. `deploy-staging` runs first. Staging values do not change, so that job should go Healthy without a prune. `deploy-production` then waits for an Engineers reviewer. That job sets `disable-sync-validation: true`, so a green production job can finish while Argo CD is still OutOfSync. Do not treat the green job as the prune.

After the production promote, open https://argocd.production.calendly-internal.com/applications/argocd/booking. The application Sync button is on the application page, not the Delete button on the Deployment row.

1. Click Sync on the `booking` application.
2. Check Prune.
3. Synchronize.

Then the list view should have no row whose kind is `apps/Deployment` and whose name is `booking-main`. The Rollout row stays, Healthy. A public page on `calendly.com` still shows selectable days.

This values change may or may not start a new canary pause. The default steps, 20% of whatever the HPA wants, then a 60 second pause, are proved by the next production deploy that changes the pod template. With the floor at 20, 20% is about 4 pods. There is no analysis block, so a pause does not roll back on a metric.

[SCH-4994](https://calendlyapp.atlassian.net/browse/SCH-4994) baseline is done when the Deployment row is gone and the Rollout is still Healthy. [booking#500](https://github.com/calendly/booking/issues/500) starts after that.

## Rollback

Revert the pull request. That puts `transition: true` back and returns the Deployment to desired state. Do not delete the Rollout or the Deployment by hand. Leigh's description also names the later path off Rollouts entirely: `canary.revert: true`, then `canary.enabled: false`. That path is not the undo for this step. A revert does not undo a booking that already happened.

## Sources

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5115
- Parent: https://calendlyapp.atlassian.net/browse/SCH-4994
- [Canary Migration Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5618663471/Canary+Migration+Guide) — step 2 removes `transition: true`, then Sync with Prune because auto-prune is off. Step 5 repeats that in production after the production transition is healthy. The guide's `chart/values-*.yaml` paths are the generic layout. This repo uses `values/`.
- [Canary Reference Guide](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/5619351578/Canary+Reference+Guide) — default steps after this handoff are `setWeight: 20`, pause 60s, then analysis. Analysis is skipped when it is not configured.
- Existing pull request: https://github.com/calendly/booking/pull/499
- Prior stage, verified 2026-10-27: https://github.com/calendly/booking/pull/498
- Production Rollout: https://argo-rollouts.production.calendly-internal.com/rollouts/booking
- Production Argo CD: https://argocd.production.calendly-internal.com/applications/argocd/booking
- The one Jira comment is Automation for Jira asking for a dev-tested or needs-qa-review label. The ticket already has `dev-tested`.

## Review

No outside team reviews this diff. Production deploy protection is separate: the `production` environment requires an Engineers reviewer at promote time. Request the same squad reviewer used on #498. No Slack post is required for the diff. Notifications after deploy go to `#alerts-scheduling-and-availability`.
