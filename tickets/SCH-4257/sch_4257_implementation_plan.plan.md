---
name: SCH-4257 enter details alignment
overview: On desktop with booking_app_redesign on, the 400px Enter details block sits on the left of the right column. Center that block. Leave field text left-aligned.
branch: SCH-4257-center-enter-details
pullRequest: https://github.com/calendly/booking/pull/526
started: 2026-10-17
pullRequestOpened: 2026-10-17
released: 2026-10-19
todos:
  - id: center-step
    content: On the Panels DetailsForm in enter_details/index.js, add auto horizontal margins when booking_app_redesign is on and the viewport is desktop
    status: pending
  - id: details-test
    content: Assert desktop redesign centers that form, desktop flag-off stays left, and the phone heading stays centered
    status: pending
isProject: false
---

# Center the Enter details step

Shareable implementation note for [SCH-4257](https://calendlyapp.atlassian.net/browse/SCH-4257) (ET Scheduling · 3 SP · Ready For Dev · P2 · Low impact).

SLA Breach Date is not set. Parent: [SCH-3792](https://calendlyapp.atlassian.net/browse/SCH-3792).

Repo: `calendly/booking`. Local checkout: `/Users/somo.banerjee/Documents/projects/SCH-4257/booking`, a fresh clone of `main` at `71b03e1c`. `frontend/components/enter_details/index.js` and `frontend/components/details_form.js` match that commit.

Branch: `SCH-4257-center-enter-details`

Pull request: https://github.com/calendly/booking/pull/526

Started: 2026-10-17

Pull request opened: 2026-10-17

Released: 2026-10-19

Constraint: `booking_app_redesign`. Desktop is `isDesktopUp` (min-width 1101). Phone is `isPhone` (max-width 800). Tablet-down, which already centers this block, is max-width 1100.

## Goal

The staging screenshot is the desktop two-column page. The sidebar is visible, so the viewport is wider than 1100px. Back, the title, the time line, the fields, and the schedule button form one block on the left of the right column, with empty space to their right. The Figma frame shows that same block centered in the right column. Text inside the block stays left-aligned. The red lines on the Figma screenshot mark that column. The "Your schedule" pane on the right of the Figma frame is not part of the booking page.

`DetailsForm` in `frontend/components/details_form.js` is the block. It is `max-width: 400px`. It sets `margin-right: auto` and `margin-left: auto` only for `isTabletDown`. Desktop does not get those margins, so the block stays at the start of `ContentWrapper` inside `EnterDetailsMainPanel` (50% of the card).

```4:13:frontend/components/details_form.js
const DetailsForm = styled('div').attrs({
  isTabletDown: useDeviceType('isTabletDown')
})`
  max-width: 400px;

  &.isTabletDown {
    margin-right: auto;
    margin-left: auto;
  }
`
```

`Panels` in `frontend/components/enter_details/index.js` renders the back control, `EnterDetailsHeadingGroup`, and `BookingForm` inside that `DetailsForm`. `EnterDetailsHeadingGroup` sets `text-align: center` only when `isPhoneRedesign` (`booking_app_redesign` and `isPhone`). That centers the phone title and time line. It does not position the desktop block.

`RedesignLegalFooterContainer` uses `align-self: stretch` and `text-align: left`. That is the legal footer, not the form. `LinksContainer` uses `justify-content: space-between`. That is the cookie row on the flag-off tablet layout. Neither rule is the offset in the screenshot.

## Out of scope

- Time-slot alignment ([SCH-4247](https://calendlyapp.atlassian.net/browse/SCH-4247)).
- Payment, Stripe, and question-field behavior. A question field stays inside the same block.
- `frontend/components/details_form.js`. Cancellation (`frontend/components/cancellation/form/form.js`) and the email-verification step use it. Their desktop alignment stays as it is.
- The phone rule on `EnterDetailsHeadingGroup`.
- Tablet-down auto margins already on `DetailsForm`. They apply with the flag off and on. Leave them.
- Desktop with `booking_app_redesign` off. That block stays left-aligned.
- The legal footer, including `align-self: stretch` on `RedesignLegalFooterContainer`. On desktop redesign the footer stays in the sidebar.
- `@calendly/booking-kit` and `calendly/js-packages`. Do not republish booking-kit.
- The order of the back control, time summary, form, and footer.

## Code changes

In `frontend/components/enter_details/index.js` only:

1. Style the `DetailsForm` used by `Panels` (not the one in `EnterVerificationCode`). When `bookingAppRedesign` is true and `isDesktopUp` is true, set `margin-right: auto` and `margin-left: auto`.
2. Keep `max-width: 400px`. The fix is the block's position in the right column.
3. Leave `EnterDetailsHeadingGroup` as it is. Phone still gets `text-align: center`. Desktop title, time line, labels, and inputs stay left-aligned inside the block.
4. Leave `EnterDetailsMainPanel`, `ContentWrapper`, and `RedesignLegalFooterContainer` as they are.

## Tests

Jest `matchMedia` in `__tests__/support/setup.js` always reports `matches: false`, so `useDeviceType` is false unless the example mocks it. Mock `@calendly/ui/hooks/use-device-type` in the new examples. Turn the flag on with `mockFeatureFlag('booking_app_redesign')` from `@calendly/feature-check-react/testing`.

Add the examples next to the enter-details render, in `frontend/components/enter_details/__tests__/enter_details_with_instant.test.js` or a sibling file if a file-level device mock would change the examples already there. The form node is the parent of `[data-component="enter-details-heading-group"]`.

From the booking repo root:

```bash
yarn exec jest frontend/components/enter_details/__tests__/enter_details_with_instant.test.js
```

Assert:

- Flag on and `isDesktopUp`: that parent has `margin-left: auto` and `margin-right: auto` (`toHaveStyle`, same as `spotpicker/__tests__/times.test.js`).
- Flag off and `isDesktopUp`: those margins are not `auto`.
- Flag on, `isPhone`, and `isTabletDown`: `[data-component="enter-details-heading-group"]` still has `text-align: center`.

## Dev test

Run this after the pull request exists and before requesting review. The Jest command above is not this check.

Booking deploys a sandbox from `.github/workflows/sandbox-deploy.yml` when the pull request has the `staging-overlay-sandbox` label.

1. Put the `staging-overlay-sandbox` label on the pull request. Wait until that workflow finishes.
2. In the Signadot extension, select that sandbox and open staging until the sandbox banner is up. Setup is [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot).
3. Open a one-on-one booking page whose response has `booking_app_redesign` on. The proof on the page is the redesign back control in the details column (chevron and "Back"), and the legal footer in the sidebar. The circular booking-kit back button is the flag-off page.
4. Pick a time. On Enter details, set the viewport wider than 1100px. The back control, title, time line, fields, and schedule button are one block in the middle of the right column, with space on both sides. Labels stay left-aligned inside the block.
5. Reload the same page with the flag off, still wider than 1100px. The block stays on the left of the right column.
6. With the flag on, narrow to phone width (800px or less). The title and time line stay centered. The fields stay left-aligned.

## Manual check

`booking_app_redesign` is on when the details column uses the chevron "Back" link and the legal footer is in the sidebar. A flag-off screenshot does not verify this ticket.

1. Book a time on a redesigned booking page and land on Enter details.
2. At a desktop width (sidebar visible, viewport wider than 1100px), compare the block to the Figma frame on the ticket. The block is centered in the right column. The staging screenshot is the offset to compare against.
3. Labels, inputs, and error text under a field stay with the field, on the left of the block.
4. The back control, time summary, fields, and footer still work, in that order. The footer on desktop stays in the sidebar.
5. At phone width, the heading and time line stay centered.
6. With the flag off, desktop Enter details stays left-aligned in the right column.

## Sources

- [SCH-4257](https://calendlyapp.atlassian.net/browse/SCH-4257)
- Staging screenshot: `image-20260603-144216.png` (attachment 98446). Desktop two-column page, block on the left of the right column.
- Figma screenshot: `image-20260603-144232.png` (attachment 98447). Same block centered between the red guides.
- Ryan Guyer, 2026-06-03: "Enter details content is not centered in it's section."
- Slack: https://calendly.slack.com/archives/C0B4KG0R26N/p1780494725406489
