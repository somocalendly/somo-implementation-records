# SCH-4912 — Skip the leave modal after a confirmed delete

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4912
- Type: not recorded
- Story points: not recorded
- Repo: `calendly/mf-publisher`
- Branch: SCH-4912-skip-leave-modal-after-delete
- Pull request: https://github.com/calendly/mf-publisher/pull/4122

## Timeline

- Started: 2026-10-24
- Pull request opened: 2026-10-24
- Released: 2026-10-29

## What changed

After a host confirmed deleting a dirty event type in the full-page editor, the app still opened “Leave without saving?” on the way to Meeting Types. A successful delete now skips that second modal once, then the usual leave guard applies again.

## Work

- Added a one-shot leave allowance on the editor shell and checked it before the dirty guard runs (`app/src/javascripts/features/event_type_editor_crud/editor.js`).
- Passed the allowance from the header into the kebab on desktop and phone (`app/src/javascripts/features/event_type_editor_crud/components/editor_header.js`, `app/src/javascripts/features/event_type_editor_crud/components/editor_primary_actions.js`).
- Armed the allowance only after a successful delete, then navigated to Meeting Types as before (`app/src/javascripts/features/event_type_editor_crud/components/header_more_menu.js`).
- Added tests for dirty delete without a leave popup, one-shot behavior, and failed delete (`app/src/javascripts/features/event_type_editor_crud/__tests__/editor.test.js`, `app/src/javascripts/features/event_type_editor_crud/components/__tests__/header_more_menu.test.js`).
