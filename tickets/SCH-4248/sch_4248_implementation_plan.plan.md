---
name: SCH-4248 time slot radius
overview: Selectable times in booking-kit hardcode a 4px radius. Switch those corners to radiusMedium from @calendly/ui/radius. Do not edit calendly/booking in this change.
branch: SCH-4248-time-slot-radius
pullRequest: https://github.com/calendly/js-packages/pull/405
started: 2026-10-14
pullRequestOpened: 2026-10-14
released: 2026-10-21
delivery:
  - repo: calendly/js-packages
    branch: SCH-4248-time-slot-radius
    pullRequest: https://github.com/calendly/js-packages/pull/405
  - repo: calendly/booking
    branch: SCH-4248-booking-kit-canary
    pullRequest: https://github.com/calendly/booking/pull/518
todos:
  - id: radius-token
    content: In packages/booking-kit/src/spot_list/spot/styles.js, set spotButtonCss and the double-bookable spot outline to radiusMedium
    status: pending
  - id: leave-square
    content: Leave unavailable slots at border-radius 0, conflict chips at 3px, and the mutual-availability marker at 50%
    status: pending
  - id: package-checks
    content: Run booking-kit eslint, stylelint, and jest. Do not add a computed-style radius assertion
    status: pending
isProject: false
---

# Use radiusMedium on booking-kit time slots

Shareable implementation note for [SCH-4248](https://calendlyapp.atlassian.net/browse/SCH-4248) (ET Scheduling · 3 SP · Ready For Dev · P2 · Low impact).

SLA Breach Date is not set. Parent: [SCH-3792](https://calendlyapp.atlassian.net/browse/SCH-3792).

Repo: `calendly/js-packages`, package `@calendly/booking-kit`. This window is that checkout: `/Users/somo.banerjee/Documents/projects/js-packages` on `main` at `31fc14d7`, tracking `origin/main`.

Branch: `SCH-4248-time-slot-radius`

Pull request: https://github.com/calendly/js-packages/pull/405

Started: 2026-10-14

Pull request opened: 2026-10-14

Released: 2026-10-21

`@calendly/booking-kit` here is `1.0.162`. `calendly/booking` at `/Users/somo.banerjee/Documents/projects/booking` still pins `1.0.159`.

## Goal

Selectable time buttons on the one-on-one booking page are booking-kit `SpotList`. `calendly/booking` renders that list from `@calendly/booking-kit` in `frontend/components/select_date_time/spotpicker/times.js`. The corner the invitee sees is not styled in the booking app.

In `packages/booking-kit/src/spot_list/spot/styles.js` two rules hardcode `border-radius: 4px`:

- `spotButtonCss`, on the shared `SpotButton`. `TimeButton` and `ConfirmButton` both render through it, so one token covers the time and the confirm control.
- `spotDoubleBookableCss`, the outline on the double-bookable `Spot` wrapper. It has to use the same token or the outline corner no longer matches the button.

`radiusMedium` from `@calendly/ui/radius` is `var(--border-radius-0-5x, 0.5rem)` (8px). Booking-kit depends on `@calendly/ui` `11.0.4`, which exports that name. The hardcoded 4px is the `radiusXsmall` step, not this token.

## Out of scope

- `calendly/booking`. The ticket says this pull request does not change it. `frontend/components/select_date_time/spot_button.js` and the poll vote button in `flat_spotpicker/invitee_count_time_button/mobile_invitee_count_time_button.js` still hardcode 4px. Those paint the flat spotpicker (polls), not `SpotList`. They are a follow-up in `calendly/booking`.
- Bumping the booking pin. The booking page will not show this until booking-kit is published and a separate booking change moves off `1.0.159`.
- Unavailable slots. The ticket groups unavailable outlines with the 4px corner. In `styles.js` they are already square: `unavailableTimeButtonCss` sets `border-radius: 0`, which overrides `spotButtonCss`, and `spotUnavailableCss` has a dashed border and no radius. Leave both square. Conflict-code chips stay at `border-radius: 3px`.
- The mutual-availability marker in `spot/time_button.js` (`border-radius: 50%`).
- Calendar day cells (`border-radius: 50%`).
- `padding: 13px 10px` on `spotButtonCss`.

## Code changes

In `packages/booking-kit/src/spot_list/spot/styles.js`:

1. Import `radiusMedium` from `@calendly/ui/radius`.
2. Set `border-radius` on `spotButtonCss` and on `spotDoubleBookableCss` to `radiusMedium`.
3. Do not add a radius to `spotUnavailableCss` or replace the `0` on `unavailableTimeButtonCss`.

In `scripts/extract-css-from-built.js`, load `@calendly/ui` radius tokens the same way spacing tokens are loaded. `radiusMedium` must be written as `var(--border-radius-0-5x, 0.5rem)`. The extractor's kebab-case fallback emits `var(--radius-medium)`, which `@calendly/ui` does not define.

## Tests

No booking-kit test asserts `border-radius`. `styles.js` is marked `istanbul ignore file`. Do not add a computed-style assertion; Jest does not prove a Linaria class painted 8px.

From `packages/booking-kit`:

```bash
yarn eslint .
yarn stylelint . --id
yarn jest
```

`spot/__tests__/time_button.test.js` should still pass. It checks rendered time text, not the corner.

## Manual check

This pull request does not change the booking page. Do not treat a staging booking page, including one with `booking_app_redesign` on, as proof. Poll slots stay on the booking-app 4px button until the follow-up.

After booking-kit is published and booking bumps the pin, a separate check can compare a one-on-one time and its confirm control to Figma node `2147-16263`, including hover and focus. Day cells stay circular. A poll time still uses the old radius until the booking follow-up.

## Sources

- [SCH-4248](https://calendlyapp.atlassian.net/browse/SCH-4248). The description names `calendly/js-packages` and says not to edit `calendly/booking`.
- Figma: https://www.figma.com/design/0wfMbVupCg2cegPl2qNNTa/Booking-Page-Customization?node-id=2147-16263
- Slack: https://calendly.slack.com/archives/C0B4KG0R26N/p1780425177845369
- `packages/booking-kit/src/spot_list/spot/styles.js` on `calendly/js-packages` `main` (`31fc14d`)
- `@calendly/ui` `11.0.4` `radius.js`, where `radiusMedium` is 8px
- `calendly/booking` `package.json` pin `@calendly/booking-kit` `1.0.159`, and `frontend/components/select_date_time/spotpicker/times.js` importing `SpotList`
