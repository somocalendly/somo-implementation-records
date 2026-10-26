# SCH-4832 — Let a draft event type URL be edited before publish

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4832
- Story points: 2
- Repo: `calendly/mf-publisher`
- Branch: SCH-4832-pre-publish-et-url
- Pull request: https://github.com/calendly/mf-publisher/pull/4103

## Timeline

- Started: 2026-10-21
- Pull request opened: 2026-10-21
- Released: 2026-10-23

## What changed

In the new event type editor, a draft only showed a read-only booking URL preview under the meeting name. There was no way to edit the slug before publish, and blur did not check whether the generated slug was already taken. Hosts can now edit the URL before publish, see the slug update as they type the meeting name, and get the next free slug from the server when they leave the name field. A slug they typed themselves is what gets saved.

## Work

- Let drafts use the same URL edit control as published event types, with a live preview that mirrors the meeting name until the slug is customized (`app/src/javascripts/features/event_type_editor_crud/tabs/meeting_details/meeting_name_field.js`).
- On meeting name blur, fetch the next available slug from the existing internal endpoint when the user has not customized the URL, and keep customized slugs from being overwritten by name changes (`app/src/javascripts/features/event_type_editor_crud/tabs/meeting_details/meeting_name_field.js`).
- Shared draft slug fetch and validation helpers used by the meeting name field and create flow (`app/src/javascripts/features/event_type_editor_crud/draft_slug.js`).
- On publish, send a customized slug when it passes the existing checks, otherwise send the slug returned for the current name, and block create when a customized slug is blank, invalid, or reserved (`app/src/javascripts/features/event_type_editor_crud/form/root_form.js`).
- Added tests for the draft URL preview, blur uniqueness, stale responses, and customized slugs (`app/src/javascripts/features/event_type_editor_crud/tabs/meeting_details/__tests__/meeting_name_field.test.js`).
- Added tests for create payload slug handling and reserved-slug validation on publish (`app/src/javascripts/features/event_type_editor_crud/form/__tests__/root_form.test.js`).
