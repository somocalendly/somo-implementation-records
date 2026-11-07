---
name: SCH-5044 Exit tracking for the full-page ET editor
overview: Fire the existing server-side `Event Type Editor Interaction` event with `action: exit` when a user actually leaves the full-page event type editor, with `section: header` for the back arrow and `section: unsaved_changes_modal` for Discard, and never for Continue editing, X, Save and leave, Back to menu, tab switches, or browser back without a modal.
branch: SCH-5044-et-editor-exit-tracking
pullRequest: https://github.com/calendly/mf-publisher/pull/4155
started: 2026-10-28
pullRequestOpened: 2026-10-28
released: 2026-11-07
delivery:
  - repo: calendly/mf-publisher
    branch: SCH-5044-et-editor-exit-tracking
    pullRequest: https://github.com/calendly/mf-publisher/pull/4155
issueType: Story
storyPoints: 2
todos:
  - id: surface-on-exit
    content: Add an optional `onExit` callback to `FullPageSurface` that fires only when its own back button starts the navigation to `returnTo` and `onBeforeClose` lets it through.
    status: pending
  - id: exit-tracker
    content: Add `trackEditorExit` to `event_type_editor_crud/analytics.js` and a gated `useTrackEditorExit` hook, emitting `action: exit`, the section, and `editor_version: full_page_editor`.
    status: pending
  - id: wire-editor
    content: In `editor.js`, pass `onExit` to the surface for `section: header`, and split the leave popup's Discard from Save and leave so only Discard tracks `section: unsaved_changes_modal`.
    status: pending
  - id: tests
    content: Extend `FullPageSurface.test.tsx`, `analytics.test.js`, `editor.test.js`, and add `use_track_editor_exit.test.js` for every tracked and untracked exit path in the acceptance criteria.
    status: pending
isProject: false
---

# SCH-5044 — Track exits from the full-page event type editor

[SCH-5044](https://calendlyapp.atlassian.net/browse/SCH-5044) (ET Scheduling · Story · 2 SP · Ready For Dev · P1 · Low impact). Repo: `calendly/mf-publisher`. Checkout: `/Users/somo.banerjee/Documents/projects/SCH-5044/mf-publisher` on `main` (`1ef6bf17b7`).

Branch: `SCH-5044-et-editor-exit-tracking`
Pull request: https://github.com/calendly/mf-publisher/pull/4155

Started: 2026-10-28. Pull request opened: 2026-10-28. Released: 2026-11-07.

This file is the design document and the implementation plan. The event is the existing analytics v2 `Event Type Editor Interaction` v1, sent through `useTrack` to `POST /api/analytics/track` on the monolith. The data contract already allows `action: exit`, `section: header`, `section: unsaved_changes_modal`, and `editor_version` on this event ([data-contracts #416](https://github.com/calendly/data-contracts/pull/416), merged 2026-10-26, DP-7290 Done). The monolith's generated copy of that schema has not been re-synced yet; that sync is owned by Product Analytics and is the one external dependency (see `## Review`). The editor itself is behind the `sa_et_crud_single_host` gate and the `sa_et_crud_single_host_experiment` experiment; the analytics helpers already read both without logging exposure.

## Goal

Research wants a Sprig survey when a user leaves the new editor ([DP-7200](https://calendlyapp.atlassian.net/browse/DP-7200), blocked by this ticket). Nothing fires today when the editor is exited. The editor's shell is `FullPageSurface` (`app/src/javascripts/features/app_shell/shell_layout/full_page_surface/FullPageSurface.tsx`): its back button does `history.push(returnTo)`, and a `history.block` registered by the surface asks the editor's `onBeforeClose(location)` whether to allow the transition. In `editor.js`, `EditorSurfaceFrame.onBeforeClose` allows tab pushes under `basePath/`, allows the small-tier return to the section menu (`pathname === basePath`), allows Publish's redirect, dismisses an open section overlay instead of leaving, and otherwise returns `!dirtyRef.current`. A blocked exit calls `onBlocked`, which stores the `resume` function as `pendingLeave` and opens `LeaveEditorConfirmationPopup`. That popup's Discard button and its Save and leave button both end in the same `onLeave`, which resumes the blocked navigation.

Two things make a naive "track when `onBeforeClose` allows an exit" wrong, and both are spelled out in the acceptance criteria:

- Browser back with no unsaved changes passes through the same `history.block` as the back arrow, and AC 6 says it must not fire. Only the surface knows whether its button started the transition.
- Discard and Save and leave share `onLeave`. AC 4 says Save and leave must not fire `exit`, so the popup has to tell them apart.

## Design

Three small pieces, all in `mf-publisher`.

1. `FullPageSurface` gains an optional `onExit?: () => void`. It fires only when the surface's own back button started the navigation to `returnTo` and the transition was not blocked. It does not fire when `onBack` overrides the button (the small-tier Back to menu), when `onBeforeClose` returns false, or for any transition that did not come from the button (browser back, tab pushes, redirects). Implementation: `handleBack` sets a ref before `history.push(returnTo)` and clears it after; the `history.block` callback reads the ref after `onBeforeClose` allows the transition and calls `onExit`. When no `onBeforeClose` is registered there is no block, so `handleBack` calls `onExit` directly after the push. The button, its `aria-label` (`Close` without `onBack`, `Back` with it), and every existing prop keep their behavior.

   Why here: the surface owns the back button (ADR-0002 in `features/app_shell/shell_layout/0002-full-page-surface-routes.md`: "Platform owns the back button (exit control with accessible label)"). The editor cannot learn that the button was the initiator from `location` alone, and overriding `onBack` from the editor would switch the label from `Close` to `Back` on every tier.

2. `event_type_editor_crud/analytics.js` gains `trackEditorExit({ track, section })`, which sends `Event Type Editor Interaction` with `properties: { action: 'exit', section, editor_version: EDITOR_VERSION }`. A new `use_track_editor_exit.js` wraps it exactly like `use_track_editor_tab_click.js`: `useTrack()` plus `useEtCrudEditorAnalyticsEnabled()`, no exposure logging. `editor_version` is on this event only for `exit`; the tab-click and preview events keep sending what they send today (AC 7).

3. `editor.js` wires the two exits. `EditorSurfaceFrame` calls `useTrackEditorExit()`, passes `onExit={() => trackEditorExit({ section: 'header' })}` to `FullPageSurface`, and splits the popup's callbacks: `onDiscard` tracks `section: 'unsaved_changes_modal'` and then runs the stored `pendingLeave`; `onSaveAndLeave` runs `pendingLeave` without tracking. `LeaveEditorConfirmationPopup` takes `onDiscard` and `onSaveAndLeave` instead of a single `onLeave`; `onClose` (X and Continue editing) is unchanged and tracks nothing.

Rejected:

- Tracking inside `onBeforeClose` whenever it allows a path outside the editor: fires on browser back with a clean form (AC 6) and cannot tell the back arrow from any other programmatic push out of the editor (for example the delete flow).
- Passing `onBack` from the editor for every tier to intercept the click: changes the exit button's accessible label from `Close` to `Back` and bypasses the surface's own `history.block` wiring.
- Firing `exit` from the browser `popstate` or `beforeunload`: fires when the user stays (modal shown) or closes the tab, both of which the ticket excludes.
- Adding `event_type` to `dependencies`/`entities`: the existing `Event Type Editor Interaction` calls from this editor send properties only, and a draft has no persisted id. Not asked for.

Blast radius: `FullPageSurface` has one consumer, `editor.js` (`rg "<FullPageSurface" app/src`). `LeaveEditorConfirmationPopup` has one consumer, `editor.js`. `trackEditorTabClick`, `trackEditorPreviewInteraction`, `useSaveTracking`, and `useTrackEditorCopyLink` are untouched. No copy, no styling, no routing change. Classic editor untouched.

## Out of scope

- The Sprig trigger itself ([DP-7200](https://calendlyapp.atlassian.net/browse/DP-7200)); Research captures the network response of this event.
- Re-syncing the monolith's generated schema JSON. `calendly/.cursor/rules/analytics-v2-schema-generated.mdc` forbids editing `engines/backend/app/services/analytics/v2/schema/*.json`; Product Analytics runs that sync from `data-contracts`.
- `legacy-calendly-monolith-backend-tp` in `data-contracts` also defines this event without the new values. The monolith's schema and this frontend path use `calendly-analytics-service-tp`, which already has them. Nothing to change.
- Exits that are not the back arrow or Discard: the `?et_editor=classic` redirect, deleting the event type from the header menu, closing the tab, and Publish's redirect to the created event type. None is tracked.
- Adding `editor_version` to the existing tab-click and preview events, or `source_page` attribution. AC 7 keeps existing actions as they are.
- A Storybook story. The editor has no stories (`find features/event_type_editor_crud -name "*.stories.*"` is empty) and this change renders nothing a person can see.
- Updating ADR-0002's `Public API` block. It already omits `onBlocked`, `className`, and `bodyClassName`; it is not maintained as the prop reference.

## Code changes

All paths under `app/src/javascripts/`. Base is `main` at `1ef6bf17b7`.

1. `features/app_shell/shell_layout/full_page_surface/FullPageSurface.tsx` (owner `@calendly/core-ux`)
   - Add `onExit?: () => void` to `FullPageSurfaceProps`, documented as: called when the back button's navigation to `returnTo` goes through; not called when `onBack` is set or when `onBeforeClose` blocks.
   - Hold `onExit` in a ref like `onBeforeClose` and `onBlocked`. Add `isBackButtonNavigationRef`.
   - `handleBack`: when `onBack` exists, call it and return (unchanged). Otherwise set the ref, `history.push(returnTo)`, clear the ref in `finally`; when no `onBeforeClose` is registered (so no block exists), call `onExit` after the push.
   - In the `history.block` callback, after `onBeforeClose` allows the transition and before `return undefined`, call `onExit` when the ref is set.
   - Nothing else changes: `aria-label`, `returnTo`, `onBack`, `onBeforeClose`, `onBlocked`, header/body/footer.

2. `features/event_type_editor_crud/analytics.js` (owner `@calendly/scheduling-and-availability`)
   - Import `EDITOR_VERSION` from `./constants`.
   - Export `EXIT_SECTIONS = { header: 'header', unsavedChangesModal: 'unsaved_changes_modal' }` and `trackEditorExit({ track, section })` that tracks `TRACKING_EVENT_NAMES.EVENT_TYPE_EDITOR_INTERACTION` with `{ action: 'exit', section, editor_version: EDITOR_VERSION }`.
   - `trackEditorTabClick` and `trackEditorPreviewInteraction` stay as they are.

3. `features/event_type_editor_crud/use_track_editor_exit.js` (new)
   - Same shape as `use_track_editor_tab_click.js`: `useTrack()`, `useEtCrudEditorAnalyticsEnabled()`, return `(params) => { if (!isEnabled) return; trackEditorExit({ track, ...params }) }`.

4. `features/event_type_editor_crud/components/leave_editor_confirmation_popup.js`
   - Props become `{ isOpen, isLive, eventTypeId, onClose, onDiscard, onSaveAndLeave }`. Discard button `onClick={onDiscard}`; Save and leave `onClick={() => handleSubmit(onSaveAndLeave)}`; Continue editing and the X keep `onClose`. Copy, test ids, and the `canSaveAndLeave` rule are unchanged.
   - Update the header comment that describes `onLeave`.

5. `features/event_type_editor_crud/editor.js`
   - `EditorSurfaceFrame`: `const trackEditorExit = useTrackEditorExit()`. Replace `onLeave` with `resumePendingLeave` (same body), add `onDiscard = () => { trackEditorExit({ section: EXIT_SECTIONS.unsavedChangesModal }); resumePendingLeave() }` and `onHeaderExit = () => trackEditorExit({ section: EXIT_SECTIONS.header })`.
   - `<FullPageSurface ... onExit={onHeaderExit}>`; `onBack={showBackToMenu ? onBackToMenu : undefined}` stays, so small-tier Back to menu never reaches `onExit`.
   - `<LeaveEditorConfirmationPopup ... onDiscard={onDiscard} onSaveAndLeave={resumePendingLeave}>`.
   - `onBeforeClose`, `onBlocked`, `handleSelectTab`, and `onSelectTab` (which tracks tab clicks) are unchanged.

6. Tests (see `## Testing`): `features/app_shell/shell_layout/full_page_surface/__tests__/FullPageSurface.test.tsx`, `features/event_type_editor_crud/__tests__/analytics.test.js`, `features/event_type_editor_crud/__tests__/editor.test.js`, new `features/event_type_editor_crud/__tests__/use_track_editor_exit.test.js`.

Procedure steps this change satisfies:

- "Adding new events or properties" in [Automations in the ET editor — implementation and analytics plan](https://calendlyapp.atlassian.net/wiki/spaces/SA/pages/6385401875) and `calendly/.cursor/rules/analytics-v2-schema-generated.mdc`: the contract is updated first in `data-contracts` (done, #416), the monolith JSON is generated from it (pending, Product Analytics), the event name already exists in `TRACKING_EVENT_NAMES`, and the frontend is implemented with scoped Jest tests. No hand edit to the monolith schema.
- `mf-publisher/.cursor/BUGBOT.md` and the same analytics plan page: analytics reads of `sa_et_crud_single_host` and `sa_et_crud_single_host_experiment` stay untracked (`useNewCode(gate, false)`, `useVariant(experiment, false)`); the new hook reuses `useEtCrudEditorAnalyticsEnabled` and logs no exposure.
- [Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269): the pull request changes a Core UX shell path and a Scheduling & Availability product path, so both teams are requested; the shell change is generic (a callback), carries no feature knowledge, and keeps the platform-owned button and label.
- `docs/TESTING.md` in this clone: tests use test ids (`full-page-surface-exit`, `leave-editor-confirmation-discard`, `leave-editor-confirmation-save-and-leave`, `leave-editor-confirmation-continue-editing`) and mock boundaries (`react-router-dom`, `useTrack`), not `@calendly/ui`.

Commit message, repo style: `[SCH-5044] Track exits from the full-page event type editor`.

## Testing

### Local (agent runs before Gate 2; any engineer can run on a checkout)

From `app/`:

```bash
yarn test \
  src/javascripts/features/app_shell/shell_layout/full_page_surface/__tests__/FullPageSurface.test.tsx \
  src/javascripts/features/event_type_editor_crud/__tests__/analytics.test.js \
  src/javascripts/features/event_type_editor_crud/__tests__/use_track_editor_exit.test.js \
  src/javascripts/features/event_type_editor_crud/__tests__/editor.test.js
yarn lint:es <changed files>
```

`FullPageSurface.test.tsx` asserts (with `mockPush` wired to invoke the last `history.block` callback, the way history does):

- Back click with `onBeforeClose` returning true calls `onExit` once and pushes `returnTo`.
- Back click with `onBeforeClose` returning false does not call `onExit`.
- Back click with `onBack` set calls `onBack` and not `onExit`.
- Back click with no `onBeforeClose` calls `onExit`.
- Invoking the block callback for a non-button transition (browser back) with `onBeforeClose` true does not call `onExit`.
- Existing label tests still pass: `Close` without `onBack`, `Back` with it.

`analytics.test.js` asserts `trackEditorExit` sends `Event Type Editor Interaction` with `{ action: 'exit', section, editor_version: 'full_page_editor' }` for `header` and `unsaved_changes_modal`, and that the tab-click and preview expectations are unchanged.

`use_track_editor_exit.test.js` mirrors `use_track_editor_tab_click.test.js`: dispatches when the gate is on, for experiment treatment with the product-refresh shell, and for `?et_editor=new`; does not dispatch when off or control; the gate and experiment reads are untracked.

`editor.test.js` (mocking `../use_track_editor_exit` the way it mocks `../use_track_editor_tab_click`, and wiring `mockPush` to the last blocker in the exit tests) asserts each acceptance criterion:

1. Clean form, back arrow on the medium/large tier → push to the meeting-types path and `trackEditorExit({ section: 'header' })`.
2. Dirty form, blocked exit, Discard → `trackEditorExit({ section: 'unsaved_changes_modal' })` and the navigation resumes. Same assertion when the blocked transition is the browser back (block callback invoked directly, not via the button).
3. Dirty form, X or Continue editing → no `trackEditorExit`, popup closes, no push.
4. Dirty form, Save and leave (with `useFormHandleSubmit` mocked to run its callback) → navigation resumes and no `trackEditorExit`.
5. Small tier in detail mode, back arrow (Back to menu) → push to `basePath`, no `trackEditorExit`.
6. Tab switch → no `trackEditorExit` (tab-click tracking still called). Clean form, browser back (block callback invoked directly) → allowed and no `trackEditorExit`.
7. Existing tab-click tests unchanged.

Storybook: not applicable. The editor has no stories and the change has no visual state; checked `features/event_type_editor_crud` for `*.stories.*` and `docs/STORYBOOK.md`.

Running app: `mf-publisher` is a Module Federation remote loaded by the monolith page (`README.md`, "Getting the App Running Locally": `yarn start` on 3032 and a local `calendly/calendly` on 3000 with `mfe_support.enabled: true`). Clone `calendly/calendly` into this ticket folder and start it with this `mf-publisher`, open an event type in the full-page editor (the URL is `/event_types/<id>/editor/...` and the header shows the Live/Off/Draft badge; the classic editor is a side rail), and with the Network tab filtered to `analytics/track`:

- Back arrow with no changes → one POST whose body has `name: "Event Type Editor Interaction"` and `properties: { action: "exit", section: "header", editor_version: "full_page_editor" }`.
- Change the name, back arrow, Discard changes → one POST with `section: "unsaved_changes_modal"`. Repeat with the browser back button.
- Change the name, back arrow, then X; then again with Continue editing → no POST.
- Change the name, back arrow, Save and leave → no `exit` POST (an `Event Type Updated` POST is expected).
- Narrow the window below the medium breakpoint, open a section, use the header arrow to return to the menu → no POST; then the header arrow from the menu → `section: "header"`.
- Switch tabs → only the `clicked_*` POSTs that fire today.

Local caveat: the monolith validates against its generated schema in `development` (`Analytics::V2::EventService::ENVIRONMENTS_TO_VALIDATE`), and that JSON does not have `exit` until Product Analytics syncs #416. Until then the request body is the proof and the response is the validator's rejection (`event_service.js` surfaces it as a dev-only `alert`). If the monolith cannot be started, that is reported at Gate 2 as not run, and the shared-environment check below carries the running-app proof.

### Shared environment (engineer and QA, after the pull request exists)

Load the branch in a Review App or `bravo-qa` per [Review App Setup for Extracted Publisher FE](https://calendlyapp.atlassian.net/wiki/spaces/FEP/pages/4646469663): open `https://bravo-qa.calendly.com/mfe_support/config`, enter `SCH-5044-et-editor-exit-tracking` next to `mf-publisher`, save, refresh. Confirm the override took: the page's `remoteEntry.js` request path contains `mfe/mf-publisher/SCH-5044-et-editor-exit-tracking/`. Open a single-host event type in the full-page editor (gate `sa_et_crud_single_host` on for the test user, or append `?et_editor=new`; the full-page editor is the proof) and run the six Network-tab checks from the local list. Proof for each tracked case: `POST /api/analytics/track` with the body above and a 2xx response. Proof for each untracked case: no `analytics/track` request whose body has `action: "exit"`.

Then ask Anthony Nguyen (Product Analytics) to confirm `Event Type Editor Interaction` with `action = exit` arrived for that user in Segment or Amplitude (the contract tags `action`, `section`, and `editor_version` as `amplitude:event_properties`). A 2xx on the request shows the monolith accepted it; the Segment/Amplitude row shows the contract let it through.

Signadot (`.github/workflows/sandbox-deploy.yml`, label `staging-overlay-sandbox`) is available but not needed: nothing here changes a Rails response, and the Review App override already loads this branch against a staging monolith.

## Post-production

After the `mf-publisher` production deploy of this commit (Build workflow on `main`), open a single-host event type in the full-page editor in production as an allocated or gated user (the full-page editor is the proof of path), click the back arrow with no changes, and confirm the `POST /api/analytics/track` body carries `action: "exit"`, `section: "header"`, `editor_version: "full_page_editor"` with a 2xx. Repeat one Discard. Production does not run the JSON validator (`ENVIRONMENTS_TO_VALIDATE` is `development test review`), so this does not wait on the monolith schema sync. Then Anthony confirms the events in Amplitude/Segment and DP-7200 can proceed.

## Rollback

Revert the `mf-publisher` commit on `main` and let Build deploy. That stops the `exit` events. It does not remove events already sent to Segment/Amplitude, and it does not change the data contract or the monolith schema, which keep the allowed values harmlessly. If the Discard/Save-and-leave split misbehaves, the revert also restores the previous single `onLeave`.

## Sources

- https://calendlyapp.atlassian.net/browse/SCH-5044 — description, the two screenshots (back arrow; "Discard changes" in the unsaved-changes popup), the Acceptance Criteria field, and Anthony Nguyen's comment 408406 (Discard fires for both the back arrow and browser back; tab switches and browser back without a modal do not fire).
- https://calendlyapp.atlassian.net/browse/DP-7290 — "Data contract updates - New ET Editor for Sprig triggers", Done 2026-10-26.
- https://calendlyapp.atlassian.net/browse/DP-7200 — Sprig trigger story, blocked by SCH-5044.
- https://github.com/calendly/data-contracts/pull/416 — adds `exit`, `header`, `unsaved_changes_modal`, and `editor_version` to `event_type_editor_interaction__v1` in `segment/calendly-analytics-service-tp/datacontract.yaml` (read in the clone at `/Users/somo.banerjee/Documents/projects/SCH-5044/data-contracts`, `main` at `bc38d72`).
- https://github.com/calendly/calendly/blob/master/engines/backend/app/services/analytics/v2/schema/event_type_editor_interaction_1.json — current `master` still lacks those values; last synced by Anthony's "Event schema updates - 9.22.26".
- https://github.com/calendly/calendly/blob/master/.cursor/rules/analytics-v2-schema-generated.mdc — schema JSON is generated from `data-contracts`; do not edit.
- `calendly/calendly` `engines/backend/app/services/analytics/v2/event_service.rb` and `dispatch_segment_event_service.rb` — validation only in `development test review`.
- [Automations in the ET editor — implementation and analytics plan](https://calendlyapp.atlassian.net/wiki/spaces/SA/pages/6385401875) — "How instrumentation works" and "Adding new events or properties".
- [Tracking Plans, Analytics Schemas, and Data Contracts](https://calendlyapp.atlassian.net/wiki/spaces/Growth/pages/4430004358) — the contract is the source of truth; the monolith schema is synced from it.
- [Review App Setup for Extracted Publisher FE](https://calendlyapp.atlassian.net/wiki/spaces/FEP/pages/4646469663) — branch override for the shared-environment check.
- [Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269) — Core UX owns `features/app_shell/**`; mixed PRs request both teams; review non-blocking for now.
- `mf-publisher` in this clone: `CODEOWNERS` lines 45 and 136, `features/app_shell/shell_layout/0002-full-page-surface-routes.md`, `.cursor/BUGBOT.md`, `docs/TESTING.md`, `README.md`, `.github/workflows/sandbox-deploy.yml`, and commits `1d1080374a` and `819ef324f2` (the tab-click and preview instrumentation this change follows).
- Confluence queries that returned no further procedure page: `type=page AND space in (FEP, UX) AND (title ~ "app shell" OR title ~ "full-page" OR ...)` returned only the three pages above; "Need Segment Event Support?" points to "Product Analytics Operations", whose body is empty.

## Review

- Product Analytics — Anthony Nguyen (reporter; `@calendly/data-contract-owners` in `data-contracts/CODEOWNERS`). Policy from the analytics plan page and the monolith rule: contract first, then sync, then implement. The contract is done (#416). Decision needed from Anthony: run the `data-contracts` → monolith schema sync so `development`, `test`, and `review` environments accept `action: exit`; production does not validate and is not blocked. Request to post with the pull request: "SCH-5044 is implemented against data-contracts #416. The monolith copy of `event_type_editor_interaction_1.json` has not been re-synced since 9.22.26, so local and review environments reject `action: exit` until it is. Can you include it in the next sync? Production is unaffected."
- Core UX — `@calendly/core-ux`, owner of `app/src/javascripts/features/app_shell/` (`CODEOWNERS` line 45; last match for `FullPageSurface.tsx` and its test). Why: this change adds `onExit` to the platform-owned surface. Policy from [Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269): Core UX reviews shell changes; a PR touching shell and product paths requests both teams; review is non-blocking for now; the page names no Slack channel, so none is used. What satisfies it: the surface change is a generic callback with no feature knowledge, the button and `aria-label` are unchanged, and `FullPageSurface.test.tsx` covers the new prop (`## Code changes` 1, `## Testing`).
- QA — the ticket carries `needs-qa-review`; the shared-environment steps above are the QA script.
- Data Platform — DP-7200 is blocked by this ticket; the pull request link goes on DP-7200 when it opens.
- Scheduling & Availability owns every other changed path (`CODEOWNERS` line 136).
