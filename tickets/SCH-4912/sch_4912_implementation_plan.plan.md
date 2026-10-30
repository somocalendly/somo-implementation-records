---
name: SCH-4912 delete leave guard
overview: After a confirmed delete of a dirty event type in the full-page editor, let that one navigation back to Meeting Types through without the leave-without-saving modal.
branch: SCH-4912-skip-leave-modal-after-delete
pullRequest: https://github.com/calendly/mf-publisher/pull/4122
started: 2026-10-24
pullRequestOpened: 2026-10-24
released: 2026-10-29
delivery:
  - repo: calendly/mf-publisher
    branch: SCH-4912-skip-leave-modal-after-delete
    pullRequest: https://github.com/calendly/mf-publisher/pull/4122
todos:
  - id: arm-bypass
    content: Arm a one-shot leave allowance in EditorSurfaceFrame and honor it at the start of onBeforeClose.
    status: pending
  - id: thread-callback
    content: Pass the allowance through EditorHeader and EditorPrimaryActions into HeaderMoreMenu, and call it only after a successful delete.
    status: pending
  - id: tests
    content: Cover the dirty successful delete, the one-shot, and a failed delete in editor.test.js and header_more_menu.test.js.
    status: pending
isProject: false
---

# SCH-4912 — Skip the leave modal after a confirmed delete

Ticket: [SCH-4912](https://calendlyapp.atlassian.net/browse/SCH-4912). Repo: `calendly/mf-publisher`, checkout `/Users/somo.banerjee/Documents/projects/SCH-4912/mf-publisher` on `main`. This file is the design document and the implementation plan.

Branch: `SCH-4912-skip-leave-modal-after-delete`

Pull request: https://github.com/calendly/mf-publisher/pull/4122

Started: 2026-10-24. Pull request opened: 2026-10-24. Released: 2026-10-29.

The editor is the full-page CRUD shell behind `sa_et_crud_single_host` (`event_type_editor_crud/constants.js`). The triage comment wrote the gate with hyphens; the constant uses underscores. No new copy, no flag change, no package publish.

## Goal

Deleting a published event type from the full-page editor, after a pending edit, asks twice. The kebab Delete opens `DeleteEventTypeConfirmationPopup` ("Delete event type?"). Confirming runs `useDeleteEventTypeMutation`. On success, `HeaderMoreMenu` pushes Meeting Types (`header_more_menu.js` `handleDelete`).

That push is not a free navigation. `FullPageSurface` registers `history.block`. `EditorSurfaceFrame.onBeforeClose` returns false when `dirtyRef` is set and the destination is outside the editor (`editor.js`). `onBlocked` then opens `LeaveEditorConfirmationPopup`. For any persisted event type, including Off, the title is "Leave without saving?" (`leave_editor_confirmation_popup.js`, `isLive={!isCreation}`).

`dirtyRef` stays true because a pending edit is still dirty after the delete mutation. The mutation does not reset the form. The user already confirmed a destructive delete, then has to confirm leaving.

The same guard already lets one other dirty navigation through: a draft Publish redirect whose path matches `ET_CRUD_EDITOR_SAVED_PATH_PATTERN` (`editor.js`). Delete has no matching exception.

## Design

Arm a one-shot allowance in `EditorSurfaceFrame`, the component that owns `onBeforeClose`. `HeaderMoreMenu` calls it only after `deleteEventType` resolves without `result.error`, and then pushes Meeting Types as it does today.

`onBeforeClose` checks the allowance first and, when it is set, clears it and returns true. First, before the overlay and dirty branches, because the event type is already gone. An open notification drawer must not swallow that navigation. `history.block` runs synchronously inside `history.push` (`FullPageSurface.tsx`), so the push that follows the call is the navigation that consumes the allowance. A later exit sees a normal dirty form again.

The allowance is a ref, passed as a callback. It is threaded `EditorSurfaceFrame` → `EditorHeader` → `EditorPrimaryActions` (desktop header kebab) and `EditorSurfaceFrame` → `EditorHeader` → `HeaderMoreMenu` (phone, where primary actions are hidden). The footer `EditorPrimaryActions` does not render the kebab and does not need the callback.

Rejected:

- Clearing `dirtyRef` before the push. `DirtySyncer` writes the real dirty flag from `useEditorFormDirty` on render. The form is still dirty. The publish exception is a path check in `onBeforeClose`, not a dirty clear.
- Letting the leave modal open and auto-confirming it. The second modal would still appear.
- Editing `FullPageSurface`. That file is `@calendly/core-ux` in `CODEOWNERS`. The exception belongs to this editor.
- A new context. Two call sites. A callback sits next to the existing publish exception.
- Moving the `history.push` up into `EditorSurface`. `HeaderMoreMenu` already owns `meetingTypesPath` and the failure branch. The parent only arms the guard.

Blast radius: the next navigation after a successful CRUD delete. Duplicate, back, tab changes, draft Publish, and a failed delete keep today's behavior. The classic sidebar delete (`event_type_editor/.../more_dropdown.js`) uses the same mutation and does not go through this guard.

## Out of scope

- A success toast. The ticket's expected behaviour asks for one. `useDeleteEventTypeMutation` (`event_type_editor/.../name_and_color_fields/api.js`) removes the event type from list caches and shows an HTTP error on failure. It does not dispatch a success alert. The Meeting Types kebab (`schedulingDeleteEventType`) is the same: success only closes the popup (`dropdown.js`). `publisher.flash.event_types` in `en.yml` has `delete_team` and no event-type-deleted flash. Adding one is new copy and, if placed on the shared mutation, new behavior on the classic editor. The triage comment marks the toast as conditional. This plan does not add it.
- Duplicate while the form is dirty. That push stays behind the leave guard.
- Draft delete. The kebab item stays disabled when `isCreation` is true.
- Classic sidebar delete, multi-host editors, and round robin / collective, which stay on the classic editor (`is_event_type_editor_crud_eligible.js`).
- [SCH-4188](https://calendlyapp.atlassian.net/browse/SCH-4188). Won't Do. It named `event_type_editor_v4/nav_guard/use_leave_guard.js`. That path is not in this clone. The live guard is the one in `editor.js`. Leave copy and the draft-versus-live split stay as they are.
- `sa_et_crud_multi_host` and the `sa_et_crud_single_host_experiment` assignment.

## Code changes

Files:

- `app/src/javascripts/features/event_type_editor_crud/editor.js` — add the ref and callback on `EditorSurfaceFrame`. At the start of `onBeforeClose`, if the ref is set, clear it and return true. Pass the callback into `EditorHeader`. Leave the overlay, in-editor, and `ET_CRUD_EDITOR_SAVED_PATH_PATTERN` branches as they are. Leave `dirtyRef` and `LeaveEditorConfirmationPopup` as they are.
- `app/src/javascripts/features/event_type_editor_crud/components/editor_header.js` — accept the callback and pass it to `EditorPrimaryActions` and to the phone `HeaderMoreMenu`.
- `app/src/javascripts/features/event_type_editor_crud/components/editor_primary_actions.js` — accept the callback and pass it to the header `HeaderMoreMenu` only.
- `app/src/javascripts/features/event_type_editor_crud/components/header_more_menu.js` — in `handleDelete`, call the callback only when `result?.error` is absent, then `history.push(meetingTypesPath)`. A failed delete still does not navigate and does not arm the allowance.
- `app/src/javascripts/features/event_type_editor_crud/__tests__/editor.test.js` — regression below.
- `app/src/javascripts/features/event_type_editor_crud/components/__tests__/header_more_menu.test.js` — callback assertion below.

No change to `FullPageSurface`, the delete mutation, locales, or the gate.

Storybook: the repo has Storybook (`yarn storybook`). `event_type_editor_crud` has no stories. [Writing Storybook Stories in mf-publisher](https://calendlyapp.atlassian.net/wiki/spaces/FEP/pages/4628971555/Writing+Storybook+Stories+in+mf-publisher) is about a component boundary and a state the story renders on its own. This change is a one-shot answer from `history.block`. The new state is the absence of the leave popup during that push. A story would need a click sequence, and Chromatic would not show the fix. No story file.

No new strings, so the Crowdin flow does not apply. Checked `app/locales/en.yml` for an existing delete-success flash and did not find one.

## Testing

### Local

From `mf-publisher/app`:

```bash
yarn test src/javascripts/features/event_type_editor_crud/__tests__/editor.test.js src/javascripts/features/event_type_editor_crud/components/__tests__/header_more_menu.test.js
```

`editor.test.js` today stubs `history.block` and does not run it from `history.push`, so a delete click cannot show the leave popup. The new example makes that test's `push` call the blocker registered with `mockBlock`, the way `FullPageSurface` does, and only for this example.

Assert, with `mockIsFormDirty` true and `isCreation` false:

- Successful delete pushes the Meeting Types path and does not render `leave-editor-confirmation-save-and-leave` or `leave-editor-confirmation-discard`.
- A second exit to that same path still opens the leave popup. The allowance is one shot.
- A failed delete does not push, and the next exit still opens the leave popup.

`header_more_menu.test.js` already asserts the mutation arguments (`ownerId`, `ownerType`, `eventTypeId`, `uuid`) and that failure does not push. Extend the success example so a passed allowance callback is called once before the push. Extend the failure example so the callback is not called.

Running app, before the diff is offered for commit. This is a frontend-only change, so use [Local Development Guide for Frontend Changes in mf-publisher Using Signadot and Marina](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6016827403/Local+Development+Guide+for+Frontend+Changes+in+mf-publisher+Using+Signadot+and+Marina), per-session steps 1–5: local `mf-publisher` via Docker, manifest at `https://mf-publisher.local.calendly-internal.com/manifest.json`, `mfe` Signadot sandbox, MFE Config set to `local` for `mf-publisher`. Proof the page is on the laptop build: `manifest.json` and chunks load from `https://mf-publisher.local.calendly-internal.com`, and `window.WEBPACK_PUBLIC_PATH` is that host.

On that page, proof this is the new editor and the dirty path: the URL is `/event_types/:id/editor/...`, the shell is the full page rather than the right rail, and the primary button reads "Save changes" (`publisher.event_type_editor_crud.header.save_changes`). `?et_editor=new` forces that shell. Then kebab → Delete → confirm once. The app lands on Meeting Types. "Leave without saving?" does not appear.

Also, on a dirty editor, leave with the back control and confirm "Leave without saving?" still appears. With `?et_editor=classic`, delete from the sidebar still does not use this modal.

If Marina is not available on this machine, say so and run the Jest examples only. Do not claim the page was exercised.

### Shared environment

After the pull request exists, follow [Testing UI Changes with Signadot MFEs](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/4958847019/Testing+UI+Changes+with+Signadot+MFEs):

1. Open the Signadot plugin and select the MFE sandbox.
2. Open MFE Config from the sandbox banner.
3. Enter app name `mf-publisher` and this PR branch, then Update Project Paths.
4. Repeat the dirty-delete steps above.

Proof this is the PR build: the MFE Config path is the branch name, and the editor is the full-page shell (or `?et_editor=new`). Proof of the fix: one "Delete event type?" confirmation, then Meeting Types, and no "Leave without saving?". The unchanged path is `?et_editor=classic`, or leaving a dirty editor without deleting.

Chromatic is not the shared-environment check. There is no story to snapshot.

## Post-production

After this commit is on `main` and the production `mf-publisher` publish has updated `manifest.json`. On production, with the full-page editor showing ("Save changes" after a pending edit), delete once and land on Meeting Types with no second modal. Confirm the live SHA with `DD_RUM.getInitConfiguration()?.version` (`mf-publisher-<sha>`) from [Rollback and Debugging Guide for Publisher Frontend](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/4652236830/Rollback+and+Debugging+Guide+for+Publisher+Frontend), Context, "In the Browser Console". This check waits until that publish. It is not run on the branch.

## Rollback

[Rollback and Debugging Guide for Publisher Frontend](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/4652236830/Rollback+and+Debugging+Guide+for+Publisher+Frontend), Option 2: GitHub Revert of this pull request, merge to `main`, and let the normal publish update the manifest.

That undo puts the second modal back. It does not restore event types people deleted while this code was live. It does not take down later commits.

If the publish is already hurting production and later commits must come down too, Option 1 of that guide (GCS Rollback to the last good SHA) is the on-call path. [Emergency git revert for mf-publisher](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6515687472/Emergency+git+revert+for+mf-publisher) says to page Frontend Platform on-call before that, and that a manifest restore also unpublishes every commit after the chosen SHA.

## Sources

- https://calendlyapp.atlassian.net/browse/SCH-4912 — description, parent SCH-4200, triage comment (2026-09-15). Slack source linked from the ticket was not opened.
- https://calendlyapp.atlassian.net/browse/SCH-4188 — related, Won't Do, different path.
- [New Event Type editor — architecture audit](https://calendlyapp.atlassian.net/wiki/spaces/SA/pages/6385926148/New+Event+Type+editor+architecture+audit) — leave confirmation, `dirtyRef`, gate, eligibility, zero Storybook stories in this feature.
- [Writing Storybook Stories in mf-publisher](https://calendlyapp.atlassian.net/wiki/spaces/FEP/pages/4628971555/Writing+Storybook+Stories+in+mf-publisher) — stories are for a rendered component state. None added.
- [Local Development Guide for Frontend Changes in mf-publisher Using Signadot and Marina](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6016827403/Local+Development+Guide+for+Frontend+Changes+in+mf-publisher+Using+Signadot+and+Marina) — local running-app check.
- [Testing UI Changes with Signadot MFEs](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/4958847019/Testing+UI+Changes+with+Signadot+MFEs) — shared-environment check. Read in full; the body is the four steps above.
- [Rollback and Debugging Guide for Publisher Frontend](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/4652236830/Rollback+and+Debugging+Guide+for+Publisher+Frontend) — Option 2 for this change, Option 1 if the publish is on fire.
- [Emergency git revert for mf-publisher](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6515687472/Emergency+git+revert+for+mf-publisher) — on-call path when a manifest restore would drop later commits. Not the rollback for a normal revert.
- Searches that did not change the design: "Event Type Editor leave without saving", "mf-publisher frontend testing Storybook", v4 interaction spec (the v4 folder's only page is the live status tracker, not a delete spec). [Event Type Editor Explicit Save](https://calendlyapp.atlassian.net/wiki/spaces/SA/pages/5941755918/Event+Type+Editor+Explicit+Save) is the classic right rail, not this shell.

## Review

None. `CODEOWNERS` last match for `app/src/javascripts/features/event_type_editor_crud/` is `@calendly/scheduling-and-availability`. The ticket's squad is Scheduling. `FullPageSurface` is `@calendly/core-ux` and this plan does not change it. No procedure page named an outside reviewer for this kind of bug fix. The rollback guide names Frontend Platform on-call only for an on-fire publish, not for review of this pull request.
