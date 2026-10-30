# SCH-4261 — Make Your schedule open the explainer

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4261
- Type: not recorded
- Story points: 3
- Repo: `calendly/booking`
- Branch: SCH-4261-your-schedule-popup
- Pull request: https://github.com/calendly/booking/pull/547

## Timeline

- Started: 2026-10-29
- Pull request opened: 2026-10-29
- Released: 2026-10-30

## What changed

On the redesigned booking page, “Your schedule” stayed inert when the signed-in user had no connected calendar, and the header often hid their avatar. Logged-in invitees now see their avatar on that header and can open the existing mutual-availability explainer from the keyboard or a click, with or without calendar accounts.

## Work

- Opened the mutual-availability explainer from the redesign header even when calendar accounts are empty (`frontend/components/mutual_availability/mutual_availability.js`).
- Showed the signed-in user’s avatar on the redesign schedule header whenever the panel is visible (`frontend/components/mutual_availability/mutual_availability.js`).
- Kept the header as a real button named “Your schedule” so keyboard users can reach the popup (`frontend/components/mutual_availability/mutual_availability.js`).
- Added a test that the header opens the popup and shows the avatar when calendar accounts are empty (`frontend/components/mutual_availability/__tests__/mutual_availability.test.js`).
