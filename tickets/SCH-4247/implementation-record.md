# SCH-4247 — Align the date label with the time slots

## Metadata

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4247
- Repo: `calendly/booking`
- Branch: not recorded
- Pull request: not recorded
- Parent: [SCH-3792](https://calendlyapp.atlassian.net/browse/SCH-3792)
- Squad / priority / points: ET Scheduling · P2 · 3 SP · Low impact
- Status: Released

## Timeline

- Started: not recorded
- Pull request opened: not recorded
- Released: 2026-10-12

## Story

On the redesigned booking page, the date above the time slots starts at the left edge of the column. The slots under it are inset. With `booking_app_redesign` on, tablet and desktop, that date row uses the same horizontal inset as the slot list: 8px left and 16px right, after those values are confirmed on installed booking-kit 1.0.159. A 2px gap inside the slot’s own margin is expected.

Left alone: the phone header, month-title alignment ([SCH-4255](https://calendlyapp.atlassian.net/browse/SCH-4255)), and slot corner radius ([SCH-4248](https://calendlyapp.atlassian.net/browse/SCH-4248)).

## Code

- `frontend/components/select_date_time/spotpicker/times.js`: pad `TimesHeaderRow` with `spacing0_5x` left and `spacing1x` right when that matches booking-kit 1.0.159. That row renders only when `booking_app_redesign` is on and `isTabletUp` is true.
- Keep `DateSubtitle` truncating long dates. Do not change `PhoneTimesHeaderRow`.
- `times.test.js` asserts the subtitle row’s own padding. It does not compare computed style against the slot list.

## Proof and rollback

Flag on, desktop and tablet: the date lines up with the time-slot column, the timezone stays clear of the scrollbar, and a long date truncates. Phone is unchanged.

The plan does not record that this check was run. It does not name a rollback.

## People

The plan does not name an outside reviewer. Ryan’s comment is “Date label does not align with time slots.”

## Still open

Branch and pull request are not in the plan. The production check is not recorded.

Plan: tickets/SCH-4247/sch_4247_implementation_plan.plan.md
