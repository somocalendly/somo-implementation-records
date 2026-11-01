# SCH-5150 — Keep managed event creates in the side panel

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5150
- Type: Bug
- Story points: 2
- Repo: `calendly/mf-publisher`
- Branch: SCH-5150-managed-et-create-stays-in-panel
- Pull request: https://github.com/calendly/mf-publisher/pull/4176

## Timeline

- Started: 2026-10-30
- Pull request opened: 2026-10-30
- Released: 2026-10-30

## What changed

Creating a one-on-one managed event type with the new editor on saved the event, then redirected into the full-page editor. That route 404’d and could show a dirty-form confirm even though save succeeded. Managed creates now stay in the classic side panel on Assign; normal one-on-one creates still open the full-page editor.

## Work

- Skipped the full-page editor redirect after create when the parent is a managed event type, so success uses the existing side-panel flow (`app/src/javascripts/features/event_type_editor/control_panel/sections/new_form/new_form.js`).
- Added tests that managed create does not push the editor route and still opens Assign on the organization owner (`app/src/javascripts/features/event_type_editor/control_panel/sections/__tests__/new_form.test.js`).
