# SCH-4682 — Fix description spacing and dark mode

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4682
- Story points: 3
- Repo: `calendly/mf-publisher`
- Branch: SCH-4682-description-spacing
- Pull request: https://github.com/calendly/mf-publisher/pull/4115

## Timeline

- Started: 2026-10-22
- Pull request opened: 2026-10-22
- Released: 2026-10-23

## What changed

On Meeting details in the new event type editor, the description label sat too close to the Quill editor, and in dark mode the description text and border did not match other CUI inputs. The label gap now aligns with neighboring fields, and dark mode uses the standard text and stroke tokens on the description editor.

## Work

- Aligned spacing between the Description label and the editor with other fields on the meeting details tab (`app/src/javascripts/features/event_type_editor_crud/tabs/meeting_details/meeting_details_tab.js`).
- Set dark-mode text and border styling on the Quill description field using CUI tokens (`app/src/javascripts/components/event_type_edit/description/description.js`, `app/src/javascripts/components/core/form/rich-text-field.js`).
- Added tests for dark-mode description text and border tokens on the meeting details tab (`app/src/javascripts/features/event_type_editor_crud/tabs/meeting_details/__tests__/meeting_details_tab.test.js`).
