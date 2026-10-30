# SCH-4255 — Align the month title and the selected date with the profile back link

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4255
- Type: not recorded
- Story points: 3
- Repo: `calendly/booking`
- Branch: SCH-4255-align-date-time-back
- Pull request: https://github.com/calendly/booking/pull/542

## Timeline

- Started: 2026-10-25
- Pull request opened: 2026-10-25
- Released: 2026-10-30

## What changed

From a host landing page, the profile back link left the calendar month title and the time-column date on different horizontal lines and in different type. With the redesign back link showing, both labels now share the link’s left inset and the month title’s typography, and the time slots stay aligned with the date.

## Work

- Shared the profile-back visibility check between the dates and times columns (`frontend/utils/booking_profile_back_url.js`).
- Defined the back-row height and month offset once on the spotpicker row (`frontend/components/select_date_time/spotpicker/spotpicker.js`).
- Moved the split month title to the back-link inset and sized the back row for alignment (`frontend/components/select_date_time/spotpicker/dates.js`).
- Put the selected-date label on the same line and type as the month title and moved the slot list with it (`frontend/components/select_date_time/spotpicker/times.js`).
- Added tests for the back-link, flag-off, and phone paths (`frontend/components/select_date_time/spotpicker/__tests__/dates.test.js`, `frontend/components/select_date_time/spotpicker/__tests__/times.test.js`).
