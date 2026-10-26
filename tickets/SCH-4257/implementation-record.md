# SCH-4257 — Center the Enter details step

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4257
- Story points: 3
- Repo: `calendly/booking`
- Branch: SCH-4257-center-enter-details
- Pull request: https://github.com/calendly/booking/pull/526

## Timeline

- Started: 2026-10-17
- Pull request opened: 2026-10-17
- Released: 2026-10-19

## What changed

On the redesigned booking page, the Enter details block sat on the left of the right column on desktop. It now centers in that column when `booking_app_redesign` is on and the viewport is wider than 1100px. Text inside the block stays left-aligned.

## Work

- Added horizontal auto margins on the Enter details form wrapper for desktop redesign only (`frontend/components/enter_details/index.js`).
- Left phone centering, flag-off desktop layout, and shared details form styling used by other flows unchanged (`frontend/components/details_form.js`).
- Added tests for desktop redesign centering, flag-off desktop, and phone heading centering (`frontend/components/enter_details/__tests__/enter_details_with_instant.test.js`).
