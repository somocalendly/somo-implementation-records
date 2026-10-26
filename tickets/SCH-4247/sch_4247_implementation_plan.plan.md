---
name: SCH-4247 date label alignment
overview: With booking_app_redesign on, the date label above the time slots has no horizontal inset while the spot list is padded 8px left and 16px right. Pad the times subtitle row with spacing0_5x and spacing1x.
branch: SCH-4247-align-date-label-with-slots
pullRequest: https://github.com/calendly/booking/pull/516
started: 2026-10-13
pullRequestOpened: 2026-10-13
released: 2026-10-14
delivery:
  - repo: calendly/booking
    branch: SCH-4247-align-date-label-with-slots
    pullRequest: https://github.com/calendly/booking/pull/516
todos:
  - id: align-subtitle
    content: Pad TimesHeaderRow with spacing0_5x left and spacing1x right when booking_app_redesign is on, after confirming those values on booking-kit 1.0.159
    status: pending
  - id: times-test
    content: Assert the subtitle row's own padding in times.test.js. Do not compare computed style against the list.
    status: pending
isProject: false
---

# Align the date label with the time slots

Shareable implementation note for [SCH-4247](https://calendlyapp.atlassian.net/browse/SCH-4247) (ET Scheduling · 3 SP · Ready For Dev · P2 · Low impact).

SLA Breach Date is not set. Parent: [SCH-3792](https://calendlyapp.atlassian.net/browse/SCH-3792). Ryan’s comment is “Date label does not align with time slots.” The ticket has a staging screenshot and no Figma node of its own.

Repo: `calendly/booking`. Local checkout: `/Users/somo.banerjee/Documents/projects/booking`. This repo installs `@calendly/booking-kit` **1.0.159**.

Branch: `SCH-4247-align-date-label-with-slots`

Pull request: https://github.com/calendly/booking/pull/516

Started: 2026-10-13

Pull request opened: 2026-10-13

Released: 2026-10-14

## Goal

On the redesigned booking page, the date label above the time list starts at the left edge of the times column. The slots under it do not.

The subtitle and the list are siblings in `frontend/components/select_date_time/spotpicker/times.js`. `TimesHeaderRow` has no horizontal padding. The list’s inset is not in this file. On tablet and desktop, booking-kit’s `SpotlistList` (`data-component="spotpicker-times-list"`) sets it when `isTabletUp` is true:

- `padding-left: 8px`
- `padding-right: 16px`

The right 16px is the scrollbar gutter. The stylesheet comment says the inset stays on the scroll container so the scrollbar sits on the column edge. Each slot then has `margin: 10px 2px`, so the button’s left edge is 10px from the column, not 8px. Padding the header to the list’s inset leaves that 2px. Do not add the slot margin on top. The ticket is the label versus the slot column, and the 2px is the button’s own outline gutter.

Those two padding values were read from booking-kit **1.0.160** (`dist/styles.css`, class `booking-kit_spotlist-list-is-tablet-up`). Confirm them in the installed **1.0.159** stylesheet before editing. If 1.0.159 differs, use that file’s left and right values.

```52:73:frontend/components/select_date_time/spotpicker/times.js
const TimesHeaderRow = styled('div')`
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  box-sizing: border-box;
`
// ...
const PhoneTimesHeaderRow = styled('div')`
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  width: 100%;
  padding: 12px 16px 5px;
```

`TimesHeaderRow` renders only when `booking_app_redesign` is on and `isTabletUp` is true. That is the row to pad.

## Out of scope

- Month-title alignment with the date label ([SCH-4255](https://calendlyapp.atlassian.net/browse/SCH-4255)). That is the calendar month versus this subtitle, not this subtitle versus the slots.
- Corner radius on the slots ([SCH-4248](https://calendlyapp.atlassian.net/browse/SCH-4248)).
- The phone header. `PhoneTimesHeaderRow` is `16px` on the sides. The phone `SpotlistWrapper` is `15px`, and the 8px/16px list padding does not apply below tablet. The ticket screenshot is the tablet and desktop column. Leave the phone header as it is.

## Code changes

1. In `times.js`, set `TimesHeaderRow` horizontal padding to the list inset confirmed on booking-kit 1.0.159. If that file still has 8px left and 16px right, import `spacing0_5x` and `spacing1x` from `@calendly/ui/spacing`:

```js
padding: 0 ${spacing1x} 0 ${spacing0_5x};
```

`spacing0_5x` is 8px. `spacing1x` is 16px. `spaceMedium` is 12px. Do not use it. `spaceSmall` is a deprecated alias of `spacing0_5x`.

2. Keep `DateSubtitle` as `flex: 1` and `min-width: 0` so a long date still truncates instead of pushing the timezone.

3. Do not change `PhoneTimesHeaderRow`.

## Tests

From the booking repo root:

`yarn exec jest frontend/components/select_date_time/spotpicker/__tests__/times.test.js`

`times.test.js` does not load the booking-kit stylesheet, and it mocks `SpotList`. `getComputedStyle` on `spotpicker-times-list` in that file will not be 8px and 16px. Do not assert that the row and the list have equal computed padding.

Add a case with `booking_app_redesign` on and `isTabletUp` true that reads `spotpicker-times-subtitle-row` and expects its padding to be `0 ${spacing1x} 0 ${spacing0_5x}` (or the 1.0.159 values, if those differ). The existing abbreviated-date case stays.

## Manual check

1. Turn on `booking_app_redesign`.
2. Open a booking page and select a day.
3. On desktop and tablet, the date label’s left edge lines up with the time-slot column. A 2px gap inside the slot’s own margin is expected.
4. The timezone control stays clear of the scrollbar.
5. A long localized date truncates and does not overlap the timezone control.
6. Phone is unchanged.

## Sources

- [SCH-4247](https://calendlyapp.atlassian.net/browse/SCH-4247)
- [SCH-3792](https://calendlyapp.atlassian.net/browse/SCH-3792)
- Slack: https://calendly.slack.com/archives/C0B4KG0R26N/p1780428817451479
