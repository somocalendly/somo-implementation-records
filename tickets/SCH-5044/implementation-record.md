# SCH-5044 — Track exits from the full-page event type editor

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5044
- Type: Story
- Story points: 2
- Repo: `calendly/mf-publisher`
- Branch: SCH-5044-et-editor-exit-tracking
- Pull request: https://github.com/calendly/mf-publisher/pull/4155

## Timeline

- Started: 2026-10-28
- Pull request opened: 2026-10-28
- Released: 2026-11-07

## What changed

Nothing fired when a host left the full-page event type editor, which blocked downstream Sprig survey work. Exits now send the existing `Event Type Editor Interaction` event with `action: exit` when the header back arrow actually leaves the editor or when the user discards unsaved changes; Continue editing, Save and leave, tab switches, Back to menu, and browser back without a modal still do not fire exit.

## Work

- Added an optional `onExit` callback on the full-page shell that runs only when the platform back button navigates to `returnTo` and the leave is not blocked (`app/src/javascripts/features/app_shell/shell_layout/full_page_surface/FullPageSurface.tsx`).
- Added gated `trackEditorExit` analytics and a `useTrackEditorExit` hook that send `action: exit` with the section and `editor_version: full_page_editor` (`app/src/javascripts/features/event_type_editor_crud/analytics.js`, `app/src/javascripts/features/event_type_editor_crud/use_track_editor_exit.js`).
- Split the unsaved-changes modal so Discard and Save and leave call separate handlers (`app/src/javascripts/features/event_type_editor_crud/components/leave_editor_confirmation_popup.js`).
- Wired header exit and modal discard tracking in the editor frame and left tab-click and preview instrumentation unchanged (`app/src/javascripts/features/event_type_editor_crud/editor.js`).
- Extended unit tests for the shell, analytics, editor exit paths, and the new hook (`app/src/javascripts/features/app_shell/shell_layout/full_page_surface/__tests__/FullPageSurface.test.tsx`, `app/src/javascripts/features/event_type_editor_crud/__tests__/analytics.test.js`, `app/src/javascripts/features/event_type_editor_crud/__tests__/editor.test.js`, `app/src/javascripts/features/event_type_editor_crud/__tests__/use_track_editor_exit.test.js`).
