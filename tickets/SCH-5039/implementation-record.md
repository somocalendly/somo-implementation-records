# SCH-5039 — Fire `editor_version` for every full-page editor share

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5039
- Story points: 3
- Repo: `calendly/mf-publisher`
- Branch: SCH-5039-editor-version-on-booking-page-shared
- Pull request: https://github.com/calendly/mf-publisher/pull/4031

## Timeline

- Started: 2026-10-09
- Pull request opened: 2026-10-09
- Released: 2026-10-12

## What changed

`Booking Page Shared` from the full-page event type editor only included `editor_version: full_page_editor` when an internal alpha gate was on, so experiment and query-param users never emitted it. Copy-link tracking now uses the same analytics enablement helper as other editor events, so anyone who can open that editor gets the property on header and live-modal copy link clicks.

## Work

- Switched copy-link tracking from the raw alpha gate to `useEtCrudEditorAnalyticsEnabled` so experiment treatment and `?et_editor=new` paths dispatch the event (`app/src/javascripts/features/event_type_editor_crud/use_track_editor_copy_link.js`).
- Extended hook tests to mirror tab-click analytics cases for gate, experiment with product-refresh shell, control, and query param (`app/src/javascripts/features/event_type_editor_crud/__tests__/use_track_editor_copy_link.test.js`).
