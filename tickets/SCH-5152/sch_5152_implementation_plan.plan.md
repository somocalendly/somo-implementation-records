---
name: SCH-5152 booking breakpoints
overview: Match booking's breakpoint table to the widths already baked into the booking-kit copy of @calendly/ui, so the page uses one scale until a durable one-copy fix is chosen.
branch: SCH-5152-match-kit-breakpoints
issueType: Bug
storyPoints: 2
pullRequest: https://github.com/calendly/booking/pull/555
started: 2026-10-31
pullRequestOpened: 2026-10-31
released: 2026-11-02
delivery:
  - repo: calendly/booking
    branch: SCH-5152-match-kit-breakpoints
    pullRequest: https://github.com/calendly/booking/pull/555
todos:
  - id: breakpoints
    content: Set bookingBreakpoints to the widths in the @calendly/ui copy installed with booking-kit
  - id: test
    content: Fail the suite when bookingBreakpoints and that installed copy disagree
isProject: false
---

# SCH-5152 — Match the booking page to the kit's breakpoints

[SCH-5152](https://calendlyapp.atlassian.net/browse/SCH-5152) (ET Scheduling · Bug · 2 SP · Backlog · P1 · Low impact). Repo is `calendly/booking`, checkout `/Users/somo.banerjee/Documents/projects/SCH-5152/booking` on `main` (`b55d0464`). This file is the design document and the implementation plan.

Repo: `calendly/booking`

Branch: `SCH-5152-match-kit-breakpoints`

Pull request: https://github.com/calendly/booking/pull/555

Started: 2026-10-31. Pull request opened: 2026-10-31. Released: 2026-11-02.

The booking page has no feature flag for this fix. `booking_app_redesign` can be on or off. The broken widths show up on both.

## Durable change

**Not this pull request.** This is where the booking page should end up.

The page should have one installed `@calendly/ui`. Booking-kit should read `DeviceTypeProvider` from that copy, and `bookingBreakpoints` should be the only scale. The historical table is phone through 649 and desktop from 1000. Design can choose a different table. Whatever is chosen, the kit follows it because it is the same module.

That holds only when booking-kit cannot install a second copy:

- Booking moves to CUI 11 first. Booking-kit 1.0.164 and later were built for CUI 11. The current kit declares 11.0.4. [booking#502](https://github.com/calendly/booking/pull/502) is the wrong bump: it stops at 11.0.2 and ungates Brand 2026. [SCH-5122](https://calendlyapp.atlassian.net/browse/SCH-5122) owns that theme decision.
- In `calendly/js-packages`, booking-kit moves `@calendly/ui` from a fixed dependency (`"11.0.4"`) to a peer dependency, the way `mfe-template-kit` already does. The range starts at `>=11.0.4`. The exact version stays in `devDependencies` for the kit's own build. The app then supplies the only copy, including when CUI is 12 or later.
- Booking's CI fails when more than one version of `@calendly/ui` is installed. Alert, form, hooks, clipboard, error-boundary, onetrust, and feature-check-react 2.0.14 still declare a 10.x range, so this install has to move with the app or those packages peel off onto CUI 10's 800/1100 defaults.

The 400px cap does not need its own change. In `packages/booking-kit/src/layout/styles.js`, `SpotPickerRow` applies `max-width: 400px` only on the class set when `useDeviceType('isTabletDown')` is true, and `max-width: 100%` when `wide` is set. There is no `@media` rule under `packages/booking-kit/src`. One shared hook is enough for the kit as it is today.

This pull request does none of that. It does not upgrade `@calendly/ui`, does not edit booking-kit, and does not add the duplicate-version check. Those wait until design picks the scale and the peer dependency is published. When that work lands, replace the numbers this pull request writes. Leaving them in place would make this temporary mirror the product, because one copy lets this object win over whatever defaults CUI has by then.

## Goal

Resizing a live booking page produces layout changes at widths that are not a single phone, tablet, and desktop scale. The production page in the Loom is `https://calendly.com/rpg0459/new-meeting-group-test`. October 30 on that host has no times. The time-button measurements below are November 2 (`?month=2026-11&date=2026-11-02`), resized in Chrome on 2026-10-30. A selected day does this:

| Viewport | What the page does today |
| --- | --- |
| 390 | Mobile. The calendar is gone. The heading is `Select a Time`. |
| 700 and 850 | A centered card. The calendar stays on screen, and the time list stacks under it. The row is `display: block`. Picking a time puts `Next` in the same row as that time button, under the calendar. |
| 920 | Calendar and times sit side by side inside a 680px card. The sidebar is still above the picker. |
| 1050 | The app puts the sidebar beside the picker, but the date/time row is `max-width: 400px` and centered. The calendar clips the last weekday. The time buttons and `Next` are laid out past that clip, about 69px wide each, so they are not in the panel. |
| 1280 | Sidebar beside the picker. Columns side by side. The row is not capped at 400px. Day buttons and time buttons sit inside the card. |

The month on that production page reads `October 2026`, and the date reads `Friday, October 30`. That is the pre-redesign copy. The URL has no `back=1`.

Two `useDeviceType` copies are in the production bundle `booking-676f2767e.js`, and they do not share a provider.

The booking app passes this override through `@calendly/ui` 10.18.8 in `frontend/app.js`:

- phone `maxWidth: 649`
- tablet `650–999`
- desktop `minWidth: 1000`

`@calendly/booking-kit` 1.0.170 depends on `@calendly/ui` 11.0.4. The app resolutions alias only 10.x ranges back to 10.18.8, so webpack keeps a second hook. That copy's defaults are the UI 11 viewport scale:

- phone `maxWidth: 895` (`VIEWPORT_SMALL_MAX`)
- tablet `896–1119`
- desktop `minWidth: 1120` (`VIEWPORT_LARGE_MIN`)

`useDeviceType` spreads `defaultBreakpointQueries` and then the provider value. The kit never sees the provider, so it keeps 895 and 1120. The app uses 649 and 1000. From 650 to 895 the app is a tablet and the kit is still a phone, so the calendar stays mounted and the times stack under it. From 1000 to 1119 the app is desktop and the kit is still tablet-down, so the sidebar sits beside a row the kit has capped at 400px. `wide={isTabletDown && hasSelectedDay}` in `spotpicker.js` uses the app's hook, so `wide` is already false in that band and does not lift the cap.

The kit's UI 11 dependency arrived in js-packages#399 (`439dc22b`, Kevin Glover, 2026-10-02, no Jira ticket), first published as `@calendly/booking-kit@1.0.160`. Booking stayed on 1.0.159, which still depends on UI 10, until [booking#518](https://github.com/calendly/booking/pull/518) (`8313f11d`, 2026-10-21) jumped the pin to 1.0.166. SCH-4248's commits only change the time-button radius. [SCH-4255](https://calendlyapp.atlassian.net/browse/SCH-4255) and SCH-4247 are not on this path.

## Design

Change `bookingBreakpoints` so the five keys the page uses equal the defaults in the `@calendly/ui` that booking-kit installed. Phone through 895. Tablet from 896 through 1119. Desktop from 1120. The kit's copy already uses those numbers. After this edit the app's copy does too. The two hooks still do not share a module. They describe the same widths, so the calendar, the 400px cap, `wide`, and the sidebar switch together.

What that does at the widths measured above:

- 390 stays mobile. A selected day unmounts the calendar. The heading is `Select a Time`.
- 700 and 850 become that same mobile layout. The calendar unmounts. The heading is `Select a Time`.
- 920 stays the tablet row. Calendar and times sit side by side, `wide` is on, and the sidebar stays above the picker.
- 1050 becomes that same tablet row. The sidebar stays above the picker. The row is wider than 400px, and the time buttons stay inside it.
- 1280 stays desktop. The sidebar sits beside the picker.

Every screen that reads `bookingBreakpoints` moves with this object. Phone layout lasts until 896 instead of 650. The 680px card cap in `frontend/components/layout/container.js` stays on through 1119, because it is tied to `isTabletDown`. The sidebar stays above the picker until 1120. Details, confirmation, cancellation, and the event-type list follow those edges. `routing-form-landing` keeps its own table.

This lasts until booking installs a booking-kit whose `@calendly/ui` defaults are no longer these widths. A newer CUI on main does nothing until that install. The test below fails when the installed defaults and this object disagree.

Rejected for this pull request:

- The durable change in the section above. It is the right end state and it is a larger upgrade than this production fix.
- Rolling `@calendly/booking-kit` back to 1.0.159. That restores 649/1000 and drops the time-button radius, the date-navigation hover, and the spaced am/pm.
- Forcing booking-kit onto 10.18.8. Kit 1.0.166 imports `radiusMedium` from `@calendly/ui/radius`.
- Reverting [booking#542](https://github.com/calendly/booking/pull/542) or SCH-4247.
- Changing `wide` or `DateSubtitle` in the spotpicker. The disagreement is the two tables, and `wide` already follows the app's table.

## Out of scope

The durable one-copy upgrade, the booking-kit peer dependency, and the CI check for two `@calendly/ui` versions. Ungating Brand 2026. [SCH-5122](https://calendlyapp.atlassian.net/browse/SCH-5122). SCH-4255's inset. SCH-4247's date-label alignment. The redesign flag. Routing-form landing breakpoints. The 680px card width. Editing `spotpicker.js`.

## Code changes

No procedure page owns this table. Searches are in Sources.

- `frontend/booking_breakpoints.js` — new. The object `DeviceTypeProvider` already receives, with phone `maxWidth` 895, tablet `896–1119`, tablet-down `maxWidth` 1119, tablet-up `minWidth` 896, and desktop `minWidth` 1120. A comment says these mirror the UI copy installed with booking-kit and that SCH-5152's durable change replaces them.
- `frontend/app.js` — import that object. Delete the inline `bookingBreakpoints`. Leave `DeviceTypeProvider` where it is. Do not change `BookingBodyStylesManager` or the product-refresh theme gate.
- `frontend/booking_breakpoints.test.js` — new. This file contains no breakpoint widths. Resolve `@calendly/ui/dist/breakpoints/viewport-breakpoints.js` from the directory of `@calendly/booking-kit/package.json`, read `VIEWPORT_SMALL_MAX`, `VIEWPORT_MEDIUM_MIN`, `VIEWPORT_MEDIUM_MAX`, and `VIEWPORT_LARGE_MIN`, and expect `bookingBreakpoints` to equal those four numbers on the five keys above. Today's tree fails this: the object is 649/999/1000 and the kit file is 895/1119/1120.

Leave `wide={isTabletDown && hasSelectedDay}` in `spotpicker.js` as it is. Once the app's `isTabletDown` matches the kit, `wide` stays on through 1119 and lifts the 400px cap on a selected day.

No Storybook file. This repo has no Storybook. See Testing.

## Testing

### Engineer: complete the code

`yarn test frontend/booking_breakpoints.test.js`

This repo has no Storybook. There is no `.storybook` directory and no `storybook` script. No story is added. Chromatic has nothing new to snapshot.

`booking_breakpoints.test.js` is red today because the app table and the kit's installed defaults differ. After this change it stays red when a later install changes those defaults and this object does not. The running page is the Signadot test plan.

After that test passes, CI is green, and a teammate has code-reviewed the pull request, the engineer is done. The engineer does not add the sandbox label, configure Signadot, choose a staging page, or perform the browser checks. QA setup and execution are outside this implementation plan.

## Post-production

After the booking production image that contains this commit is serving `calendly.com`, open `https://calendly.com/rpg0459/new-meeting-group-test?month=2026-11&date=2026-11-02` and resize. At 850px the calendar is gone and the heading is `Select a Time`. At 1050px the calendar and the times sit side by side, the row is wider than 400px, a time button stays inside the row, and the sidebar is above the picker. At 1280px the sidebar is beside the picker. This check runs after production. It is not part of the pull request.

## Rollback

Revert the booking commit and let the normal deploy ship the previous image. That restores phone through 649 and desktop from 1000, and the split layout returns. It does not roll back `@calendly/booking-kit`.

If the production image is already the bad one, use [Performing Rollbacks and Reverts](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/3015344227/Performing+Rollbacks+and+Reverts), section "Gitops Deployed Apps", Basic Method (Previous Commit), on the booking manifest in `argocd-gcp`. There is no migration. A rollback does not restore a visitor's in-progress selection.

## Sources

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5152. Parent epic SCH-3652. Story points 2. Issue type Bug.
- Loom: https://www.loom.com/share/26a08b875d8c4356a5ce8b4a5d098eac. Ryan Guyer, staging `https://staging.calendly.com/ryan/adsfdsaf?month=2026-10&date=2026-10-30` then production `https://calendly.com/rpg0459/new-meeting-group-test?month=2026-10&date=2026-10-30`. He describes extra breakpoints, content centering, buttons in the wrong place, and the same behavior on the pre-refresh production page. He says it should be the mobile booking experience.
- Slack thread synced onto the ticket: https://calendly.slack.com/archives/C09HBNHD3DF/p1793376255718459?thread_ts=1793376255.718459&cid=C09HBNHD3DF. Kristyn Bryan relayed a Glean guess that booking#542 caused the wrap. That guess is the lead this plan checked and rejected.
- Production bundle measured on 2026-10-30: `https://assets.calendly.com/assets/booking/js/booking-676f2767e.js`.
- Booking-kit layout source: `packages/booking-kit/src/layout/styles.js`. `SpotPickerRow` applies `max-width: 400px` on the `isTabletDown` class from `useDeviceType`, and `max-width: 100%` on the `wide` prop. No `@media` in `packages/booking-kit/src`.
- Kit defaults: `@calendly/ui` 11.0.4 `dist/breakpoints/viewport-breakpoints.js` (`VIEWPORT_SMALL_MAX` 895, `VIEWPORT_MEDIUM_MAX` 1119, `VIEWPORT_LARGE_MIN` 1120) and `dist/hooks/use-device-type/breakpoints.js` (`defaultBreakpointQueries`).
- Peer pattern, for the durable change only: `packages/mfe-template-kit/package.json` has `peerDependencies` `"@calendly/ui": ">=10.18.0"`. Booking-kit's `package.json` still has `"@calendly/ui": "11.0.4"` under `dependencies`.
- [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot). Opened for the QA path. The booking sandbox is `staging-overlay-sandbox`, not the oap-web example in step 2.
- [Performing Rollbacks and Reverts](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/3015344227/Performing+Rollbacks+and+Reverts). Outline read. Rollback uses "Gitops Deployed Apps", Basic Method (Previous Commit).
- [Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269/Updating+Core+UX+Codeowner). Summary read. This pull request does not change `js-packages`. That page does not list `calendly/booking`. It says Core UX is not a required reviewer for unrelated product code.
- Searches with no procedure for this table: Confluence `type = page AND title ~ "breakpoint" AND space = EN` (no pages). Confluence title search for dependency or bump plus text `booking` returned a draft UI 10.17.3 booking-kit token plan and older Packwerk dependency pages, not a rule for this override.

## Review

None. The squad on the ticket is Scheduling.

`calendly/booking` has no `CODEOWNERS` file. Checked the clone at `main` (`b55d0464`). The paths this change touches are `frontend/booking_breakpoints.js`, `frontend/app.js`, and `frontend/booking_breakpoints.test.js`. There is no later rule that assigns them to another team.

[Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269/Updating+Core+UX+Codeowner) does not list `calendly/booking`. It says Core UX is not a required reviewer for unrelated product code. This plan does not change `booking-kit` or `js-packages`. The page names no Slack channel for this change.
