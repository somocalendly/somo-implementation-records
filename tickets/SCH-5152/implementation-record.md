# SCH-5152 — Match the booking page to the kit's breakpoints

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5152
- Type: Bug
- Story points: 2
- Repo: `calendly/booking`
- Branch: SCH-5152-match-kit-breakpoints
- Pull request: https://github.com/calendly/booking/pull/555

## Timeline

- Started: 2026-10-31
- Pull request opened: 2026-10-31
- Released: 2026-11-02

## What changed

Resizing a live booking page jumped between layout modes at widths that did not match a single phone, tablet, and desktop scale, because the app and booking-kit each used a different `@calendly/ui` breakpoint table. The page now uses the same widths as the kit’s installed UI copy, so the calendar, sidebar, and 400px spot-picker cap change together across viewports.

## Work

- Added a shared `bookingBreakpoints` object that mirrors the viewport limits from the `@calendly/ui` copy installed with booking-kit (`frontend/booking_breakpoints.js`).
- Wired the app to import that object into `DeviceTypeProvider` and removed the inline 649/1000 table (`frontend/app.js`).
- Added a test that reads the kit’s installed viewport constants and fails when they disagree with `bookingBreakpoints` (`frontend/booking_breakpoints.test.js`).
- Left spot-picker `wide` logic and booking-kit layout code unchanged (`frontend/components/select_date_time/spotpicker/spotpicker.js`).
