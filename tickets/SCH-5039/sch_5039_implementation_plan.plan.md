---
name: SCH-5039 editor_version tracking
overview: Booking Page Shared from the full-page editor only includes editor_version full_page_editor when the internal sa_et_crud_single_host gate is on. Switch the copy-link hook to useEtCrudEditorAnalyticsEnabled so experiment and query-param users emit the same event.
todos:
  - id: switch-hook
    content: In use_track_editor_copy_link.js, replace the direct sa_et_crud_single_host check with useEtCrudEditorAnalyticsEnabled and drop the unused useNewCode and EVENT_TYPE_EDITOR_CRUD_GATE imports
    status: pending
  - id: hook-tests
    content: Extend use_track_editor_copy_link.test.js to match the tab-click analytics cases (gate, treatment plus product-refresh shell, shell off, control, et_editor=new, all off) and assert untracked Statsig reads
    status: pending
  - id: keep-call-sites
    content: Leave EditorPrimaryActions and PublishLiveModal call sites and their existing payload assertions unchanged
    status: pending
isProject: false
branch: SCH-5039-editor-version-on-booking-page-shared
pullRequest: https://github.com/calendly/mf-publisher/pull/4031
started: 2026-10-09
pullRequestOpened: 2026-10-09
released: 2026-10-12
delivery:
  - repo: calendly/mf-publisher
    branch: SCH-5039-editor-version-on-booking-page-shared
    pullRequest: https://github.com/calendly/mf-publisher/pull/4031
---

# Fire `editor_version` for every full-page editor share

Shareable implementation note for [SCH-5039](https://calendlyapp.atlassian.net/browse/SCH-5039) (ET Scheduling · 3 SP · Ready For Dev · P1 · High impact).

Parent: [SCH-4200](https://calendlyapp.atlassian.net/browse/SCH-4200).

Repo: `calendly/mf-publisher`, on current `main`. Local checkout used for this plan: `/Users/somo.banerjee/Documents/projects/mf-publisher` (branch `main`, matching `origin/main`).

## Goal

`Booking Page Shared` v2 from the full-page Event Type Editor must include `editor_version: 'full_page_editor'` for every org that can actually open that editor, not only Calendly’s internal org.

Today the property shows up in production only for the internal org because copy-link tracking returns early unless the alpha gate `sa_et_crud_single_host` is on. That gate is targeted at `internal` and `pre-alpha` (`EVENT_TYPE_EDITOR_CRUD_GATE_TARGETING` in `constants.js`). External experiment users reach the same editor through `sa_et_crud_single_host_experiment` when the product-refresh app shell is on, or through `?et_editor=new`. Their copy-link clicks never dispatch.

This blocks the single-host editor experiment. Tommy called the firing paramount for measurement on the ticket; Kristyn asked to pull it into the current sprint.

## Why this hook

`useTrackEditorCopyLink` is the only product-analytics hook in `event_type_editor_crud` that still reads the alpha gate directly:

```15:19:app/src/javascripts/features/event_type_editor_crud/use_track_editor_copy_link.js
  const isEnabled = Boolean(useNewCode(EVENT_TYPE_EDITOR_CRUD_GATE, false)?.enabled)
  const trackBookingPageShared = useTrackBookingPageShared(eventType?.id)

  return useCallback((shareLocation) => {
    if (!isEnabled) return
```

These already use `useEtCrudEditorAnalyticsEnabled()`:

- `use_track_editor_tab_click.js`
- `use_track_editor_preview_interaction.js`
- `form/save_tracking.js` (`Event Type Created` / `Event Type Updated`, also sends `editor_version`)
- `form/fix_before_publishing_cta.js`

`useNewCode` inside `useEtCrudEditorGateState` and `useEtCrudEditorExperimentExposure` is allocation and exposure, not a tracking drop. Leave those alone.

The helper already encodes the allocation model from [SCH-4984](https://calendlyapp.atlassian.net/browse/SCH-4984) (Done):

```20:29:app/src/javascripts/features/event_type_editor_crud/use_et_crud_editor_analytics_enabled.js
export const useEtCrudEditorAnalyticsEnabled = () => {
  const isProductRefreshAppShell = useProductRefreshAppShell()
  const gateOn = Boolean(useNewCode(EVENT_TYPE_EDITOR_CRUD_GATE, false)?.enabled)
  const experimentVariant = useVariant(EVENT_TYPE_EDITOR_CRUD_EXPERIMENT, false)
  const editorVariant = readEtEditorQueryParam(window.location.search)

  return gateOn
    || isEtCrudEditorExperimentTreatmentForShell(experimentVariant, isProductRefreshAppShell)
    || editorVariant === ET_EDITOR_VARIANTS.NEW
}
```

Enabled when any of these is true:

1. `sa_et_crud_single_host` is on (internal / pre-alpha). This is the path that works in production today.
2. `sa_et_crud_single_host_experiment` resolves to a treatment group (any non-empty group other than `control`, `on`, or `off`) **and** `useProductRefreshAppShell()` is true.
3. The URL has `?et_editor=new`.

Treatment with the product-refresh shell off stays off. That matches the editor itself: [SCH-4984](https://calendlyapp.atlassian.net/browse/SCH-4984) fails closed to the classic editor unless the refresh shell is actually on.

`useNewCode` and `useVariant` are called with `false`, so this read does not log a Statsig exposure. Exposure stays in `useEtCrudEditorExperimentExposure`. Keep that second argument.

## Call sites (no edits)

Both copy-link buttons already go through this one hook. Fixing the hook covers both share locations from [SCH-4851](https://calendlyapp.atlassian.net/browse/SCH-4851) (Done):

| Surface | File | `share_location` |
| --- | --- | --- |
| Header and footer Copy link | `components/editor_primary_actions.js` (`CopyLinkSplit`) | `et_editor` |
| “You’re live” modal Copy link | `components/publish_live_modal.js` (`LiveShare`) | `et_editor_youre_live_modal` |

Payload already matches the v2 contract, including `share_object_id` added after the SCH-4851 review. Do not change property names or values. `useTrackBookingPageShared` still injects `is_ai_notetaker_enabled` and sends version 2.

```mermaid
flowchart TD
  click[Copy link click]
  hook[useTrackEditorCopyLink]
  enabled{useEtCrudEditorAnalyticsEnabled}
  gate[sa_et_crud_single_host on]
  exp[experiment treatment and product-refresh shell]
  query["?et_editor=new"]
  drop[return without tracking]
  event["Booking Page Shared v2 editor_version full_page_editor"]
  click --> hook --> enabled
  enabled --> gate
  enabled --> exp
  enabled --> query
  gate -->|yes| event
  exp -->|yes| event
  query -->|yes| event
  enabled -->|none| drop
```

## Out of scope

- Changing `useEtCrudEditorAnalyticsEnabled`, experiment group matching, or the product-refresh shell requirement. Copy link should follow the helper the other editor events already use.
- Statsig exposure. Do not pass `true` as the tracked-read argument.
- Email, embed, preview, and single-use-link actions in the header menu and live modal. They do not call `useTrackEditorCopyLink`. SCH-4851 instrumented copy link only.
- Amplitude / Hex dashboard changes. The [investigation dashboard](https://app.hex.tech/dab85203-7799-4296-a36c-d880ef452783/app/Saetcrudsinglehostexperiment-investigation-034cYft1anlSZkdezzeBI4/latest) is the verification surface, not a code change.
- SCH-4851’s acceptance line that says the event must not fire outside `sa_et_crud_single_host`. That was true when the editor was internal-only. The experiment rollout made it the bug.

## Code change

`app/src/javascripts/features/event_type_editor_crud/use_track_editor_copy_link.js`

- Remove `useNewCode` from `@calendly/feature-check-react`.
- Remove `EVENT_TYPE_EDITOR_CRUD_GATE` from the constants import. Keep `EDITOR_VERSION`.
- Import `useEtCrudEditorAnalyticsEnabled` from `./use_et_crud_editor_analytics_enabled`.
- Replace the `isEnabled` line with `const isEnabled = useEtCrudEditorAnalyticsEnabled()`.
- Leave the callback body, dependency array, and `COPY_LINK_SHARE_LOCATIONS` as they are.

```js
export const useTrackEditorCopyLink = (eventType) => {
  const isEnabled = useEtCrudEditorAnalyticsEnabled()
  const trackBookingPageShared = useTrackBookingPageShared(eventType?.id)

  return useCallback((shareLocation) => {
    if (!isEnabled) return
    // existing Booking Page Shared payload, unchanged
  }, [eventType, isEnabled, trackBookingPageShared])
}
```

## Tests

Mirror `app/src/javascripts/features/event_type_editor_crud/__tests__/use_track_editor_tab_click.test.js` inside `app/src/javascripts/features/event_type_editor_crud/__tests__/use_track_editor_copy_link.test.js`.

`useProductRefreshAppShell` is already `jest.fn(() => false)` in `app/src/__tests__/support/setup.js`. Import that mock the same way the tab-click test does. Spy on `useNewCode` and `useVariant`. Reset the URL in `afterEach` if a case pushes `?et_editor=new`.

Keep the existing payload cases (both share locations, missing location → `location_type: 'none'`). Add:

- Gate on, shell off, no experiment: still dispatches `Booking Page Shared` v2 with `editor_version: 'full_page_editor'`. Existing tests already cover the payload; keep them.
- Experiment treatment (`variant_1` via `mockFeatureCheck`) **and** `useProductRefreshAppShell` true, gate off: dispatches. Assert `useNewCode(EVENT_TYPE_EDITOR_CRUD_GATE, false)` and `useVariant(EVENT_TYPE_EDITOR_CRUD_EXPERIMENT, false)`.
- Same treatment with the shell mock left `false`: does not dispatch.
- Experiment `control`, gate off: does not dispatch.
- `window.history.pushState({}, '', '/?et_editor=new')` with the gate off and no treatment: dispatches.
- Gate off, no experiment, no query param: does not dispatch. The current “gate is off” test stays, and it now also depends on the shell mock defaulting to false.

Do not add component-level experiment cases. `editor_primary_actions.test.js` and `publish_live_modal.test.js` already mock the gate on and assert the full v2 payload for `et_editor` and `et_editor_youre_live_modal`. The header “gate off → no track” case still holds because the shell mock defaults to false and those tests do not set the experiment or `?et_editor=new`.

Run:

```bash
yarn jest app/src/javascripts/features/event_type_editor_crud/__tests__/use_track_editor_copy_link.test.js app/src/javascripts/features/event_type_editor_crud/components/__tests__/editor_primary_actions.test.js app/src/javascripts/features/event_type_editor_crud/components/__tests__/publish_live_modal.test.js
```

Use whatever test script `mf-publisher` already uses if `yarn jest` is not the local entry (`package.json` at the repo root).

## Manual check

On staging, as an org that is **not** the Calendly internal org and is in the experiment treatment with the product-refresh shell:

1. Open a single-host event type in the full-page editor.
2. Click Copy link in the header. Expect `Booking Page Shared` v2 with `share_location: et_editor`, `share_method: copy_link`, `share_object: event_type`, `editor_version: full_page_editor`.
3. Publish and click Copy link in the you’re-live modal. Same event with `share_location: et_editor_youre_live_modal`.

Optional: `?et_editor=new` on an account outside the gate and outside treatment should emit the same property. An account in experiment control, or treatment without the refresh shell, should not be in this editor and should not emit it from these buttons.

Production confirmation is the Hex dashboard linked on the ticket: non-internal orgs start appearing with `editor_version = full_page_editor`.

## Sources

- [SCH-5039](https://calendlyapp.atlassian.net/browse/SCH-5039) — this bug. Slack thread linked from the description.
- [SCH-4851](https://calendlyapp.atlassian.net/browse/SCH-4851) — original header and modal copy-link instrumentation. Done. Its “gate only” acceptance criterion is stale.
- [SCH-4984](https://calendlyapp.atlassian.net/browse/SCH-4984) — experiment counts only when the product-refresh shell is on. Done. That is why the analytics helper, not the raw gate, is the right check.
- [ANALYSIS-2306](https://calendlyapp.atlassian.net/browse/ANALYSIS-2306) — analytics task for post-publish copy link. Submitted.
- [ANALYSIS-2302](https://calendlyapp.atlassian.net/browse/ANALYSIS-2302) — analytics epic for the single-host editor revamp. Submitted.
- [SCH-4200](https://calendlyapp.atlassian.net/browse/SCH-4200) — parent epic, In Dev.
- [C.O.R.E.](https://calendlyapp.atlassian.net/wiki/spaces/EN/pages/4788781065/Calendly+Ownership+Registry+for+Engineering+C.O.R.E.) — Event Type Editor is Scheduling.
