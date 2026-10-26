# SCH-4247 — Align the date label with the time slots

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4247
- Story points: 3
- Repo: `calendly/booking`
- Branch: SCH-4247-align-date-label-with-slots
- Pull request: https://github.com/calendly/booking/pull/516

## Timeline

- Started: 2026-10-13
- Pull request opened: 2026-10-13
- Released: 2026-10-14

## What changed

On the redesigned booking page, the date label above the time list sat flush with the left edge of the column while the slots below were inset. On tablet and desktop, the subtitle row now uses the same horizontal padding as the spot list so the label lines up with the slot column.

## Work

- Padded the redesign times subtitle row with the same left and right spacing tokens as the spot list inset on tablet and up (`frontend/components/select_date_time/spotpicker/times.js`).
- Left the phone times header and spot list layout unchanged (`frontend/components/select_date_time/spotpicker/times.js`).
- Added a test that the subtitle row padding matches those spacing tokens when redesign is on and the viewport is tablet-up (`frontend/components/select_date_time/spotpicker/__tests__/times.test.js`).
