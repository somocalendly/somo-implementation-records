# SCH-4908 — Keyboard access for overflow submenus

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4908
- Type: not recorded
- Story points: not recorded
- Repo: `calendly/mf-publisher`
- Branch: SCH-4908-overflow-submenu-keyboard
- Pull request: https://github.com/calendly/mf-publisher/pull/4121

## Timeline

- Started: 2026-10-24
- Pull request opened: 2026-10-24
- Released: 2026-10-29

## What changed

On the new event type editor, keyboard users could not open Page language or Event type color from the header overflow menu, and the parent row lost its highlight once that list was open. Those two rows now open from the keyboard, and the parent row stays highlighted while the list is open.

## Work

- Turned Page language and Event type color into submenus the keyboard can open (`app/src/javascripts/features/event_type_editor_crud/components/page_language_menu.js`, `app/src/javascripts/features/event_type_editor_crud/components/event_color_menu.js`).
- Kept the parent row highlighted while its submenu is open (`app/src/javascripts/features/event_type_editor_crud/components/header_more_menu.js`).
- Removed the hover-only flyout hook, its tests, and the portal stamp helper (`app/src/javascripts/features/event_type_editor_crud/hooks/use_hover_open_menu.js`, `app/src/javascripts/features/event_type_editor_crud/hooks/__tests__/use_hover_open_menu.test.js`, `app/src/javascripts/features/event_type_editor_crud/components/stamp_flyout_parent_instance.js`).
- Added keyboard tests for opening each submenu, focus return, and the existing language and color commits (`app/src/javascripts/features/event_type_editor_crud/components/__tests__/header_more_menu.test.js`).
- Added stories that snapshot each submenu already open (`app/src/javascripts/features/event_type_editor_crud/components/__stories__/header_more_menu.stories.js`).
