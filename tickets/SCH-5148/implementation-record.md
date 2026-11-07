# SCH-5148 — Darken the schedule card border on hover

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5148
- Type: Bug
- Story points: 2
- Repo: `calendly/mf-publisher`
- Branch: SCH-5148-schedule-card-hover-border
- Pull request: https://github.com/calendly/mf-publisher/pull/4178

## Timeline

- Started: 2026-10-30
- Pull request opened: 2026-10-30
- Released: 2026-11-03

## What changed

On a shared schedule in the v4 Availability section, hovering the card made the border lighter instead of darker relative to the resting state. The card now uses the light standard stroke at rest and the darker medium-hover stroke when the pointer is over the clickable shared card, matching Event Type Editor v4 Figma.

## Work

- Restored the schedule card resting border to the standard stroke and applied the medium-hover stroke when the shared card is hovered (`app/src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/v4_schedule_card.js`).
- Set the resting Edit pill border to the standard stroke and left the emphasized pill on secondary-outline (`app/src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/v4_edit_availability_button.js`).
- Added Resting and Hovered Storybook stories so Chromatic can snapshot the border states without a pointer (`app/src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/v4_schedule_card.stories.js`).
- Added tests that the clickable card toggles `data-hovered` on pointer enter and leave and that locked, read-only, and custom-hours cards never hover (`app/src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/__tests__/v4_schedule_card.test.js`).
