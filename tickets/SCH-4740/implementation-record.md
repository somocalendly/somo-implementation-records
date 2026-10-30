# SCH-4740 — Confirmations tab font token

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4740
- Type: not recorded
- Story points: not recorded
- Repo: `calendly/mf-publisher`
- Branch: SCH-4740-confirmation-font-token
- Pull request: https://github.com/calendly/mf-publisher/pull/4123

## Timeline

- Started: 2026-10-25
- Pull request opened: 2026-10-25
- Released: 2026-10-30

## What changed

On the new editor Confirmations tab, “After booking…” and the schedule-another card title used the 12px label token while matching labels elsewhere in the editor use 14px. Both now use the large label token; the card description stays small.

## Work

- Set “After booking…” to `LabelText` size large on the editable and locked Confirmations tab (`app/src/javascripts/features/event_type_editor_crud/tabs/confirmation/confirmation_tab.js`).
- Set the schedule-another card title to `LabelText` size large in collapsed and expanded states (`app/src/javascripts/features/event_type_editor_crud/tabs/confirmation/schedule_another_meeting_card.js`).
- Added a Storybook story that shows both labels at the new size without extra interaction (`app/src/javascripts/features/event_type_editor_crud/tabs/confirmation/__stories__/confirmation_tab.stories.js`).
