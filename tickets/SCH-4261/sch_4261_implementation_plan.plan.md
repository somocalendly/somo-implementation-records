---
name: SCH-4261 your schedule click
overview: The redesigned Your schedule header hides the avatar and ignores clicks unless a calendar account query returns rows. Always show the avatar and open the existing explainer popup.
branch: SCH-4261-your-schedule-popup
storyPoints: 3
pullRequest: https://github.com/calendly/booking/pull/547
started: 2026-10-29
pullRequestOpened: 2026-10-29
released: 2026-10-30
delivery:
  - repo: calendly/booking
    branch: SCH-4261-your-schedule-popup
    pullRequest: https://github.com/calendly/booking/pull/547
todos:
  - id: always-clickable
    content: Open the mutual-availability popup from the redesign header without requiring calendar accounts
    status: pending
  - id: show-avatar
    content: Render the current user’s avatar on that header
    status: pending
  - id: header-test
    content: Assert the header button opens the popup and shows the avatar when calendar accounts are empty
    status: pending
isProject: false
---

# Make Your schedule open the explainer

Shareable implementation note for [SCH-4261](https://calendlyapp.atlassian.net/browse/SCH-4261) (ET Scheduling · 3 SP · Ready For Dev · P2 · Low impact).

SLA Breach Date is not set. Parent: [SCH-3792](https://calendlyapp.atlassian.net/browse/SCH-3792).

Repo: `calendly/booking`. Local checkout: `/Users/somo.banerjee/Documents/projects/booking`.

Branch: `SCH-4261-your-schedule-popup`

Pull request: https://github.com/calendly/booking/pull/547

Started: 2026-10-29. Pull request opened: 2026-10-29. Released: 2026-10-30.

## Goal

A logged-in user viewing someone else’s booking page should see their avatar on “Your schedule” and, on click, a modal titled along the lines of “Here’s how Calendly will show times you’re available,” with a “Got it!” button. The popup already exists in `frontend/components/mutual_availability/popup/index.js`.

The redesign header only becomes a button when `useGetCalendarAccountsQuery` returns at least one account, and the avatar renders only when `avatarUrl` is set:

```89:105:frontend/components/mutual_availability/mutual_availability.js
  const RedesignDesktopHeaderComponent = hasCalendarConnected ? RedesignClickableHeader : RedesignHeader
  const RedesignPhoneHeaderComponent = hasCalendarConnected ? RedesignPhoneClickableHeader : RedesignPhoneHeader
  // ...
        <RedesignPhoneHeaderComponent onClick={hasCalendarConnected ? openPopup : undefined}>
```

`RedesignScheduleHeaderContent` draws `RedesignAvatar` only when `avatarUrl` is truthy, and the info icon only when `hasCalendarConnected` is true.

Flags from the ticket: `booking_app_redesign` and `mutual_availability_sign_up_with_google`.

## Out of scope

- Conflict detection and the event list under the header.
- The non-redesign `ClickableHeader` path, except to keep its popup behavior.

## Code changes

1. On the redesign header, call `openPopup` on click even when `calendarAccounts` is empty.
2. Render the avatar from the current user whenever the mutual-availability panel is showing. Keep the info icon as the click affordance.
3. Use a real button (`RedesignClickableHeader` / `RedesignPhoneClickableHeader`) so the control is keyboard reachable. Give it an accessible name of “Your schedule”.

## Tests

From the booking repo root:

`yarn exec jest frontend/components/mutual_availability/__tests__/mutual_availability.test.js`

Add a case with the redesign flag on, calendar accounts empty, and an avatar URL. Click “Your schedule” and expect the popup. The existing times test already looks for a button named “Your schedule” in `spotpicker/__tests__/times.test.js`; keep that passing.

## Manual check

1. Enable `booking_app_redesign` and `mutual_availability_sign_up_with_google`.
2. Sign in and open another user’s live booking page.
3. “Your schedule” shows the signed-in user’s avatar.
4. Click it. The explainer modal opens. “Got it!” closes it.
5. Repeat on a phone width.

## Sources

- [SCH-4261](https://calendlyapp.atlassian.net/browse/SCH-4261)
