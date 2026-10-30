---
name: SCH-4255 date and time alignment
overview: When the redesign back link is showing, put the month title and the selected-date label on one line at the link's 32px left inset, in the month title's type, and move the time slots with the date. Leave the day grid on the kit's 19px margin so the 44px circles still fit. No booking-kit change.
branch: SCH-4255-align-date-time-back
storyPoints: 3
pullRequest: https://github.com/calendly/booking/pull/542
started: 2026-10-25
pullRequestOpened: 2026-10-25
released: 2026-10-30
delivery:
  - repo: calendly/booking
    branch: SCH-4255-align-date-time-back
    pullRequest: https://github.com/calendly/booking/pull/542
todos:
  - id: dates-inset
    content: Keep the desktop back link in the dates column and move only the split-header month title to the 32px inset
  - id: times-line
    content: Drop the date label onto that same line, with the same type and left inset, and move the time slots with it
  - id: shared-offset
    content: Define the vertical offset once, as CSS variables on the spotpicker row
  - id: tests
    content: Cover the host-landing-page path and the flag-off path with class assertions, then measure the running page at 801, 1101, and about 1280
isProject: false
---

# SCH-4255 — Align the month title and the selected date with the profile back link

[SCH-4255](https://calendlyapp.atlassian.net/browse/SCH-4255). Repo is `calendly/booking`, checkout `/Users/somo.banerjee/Documents/projects/SCH-4255/booking` on `main` (`91ccf951`). The flag is the existing `booking_app_redesign` gate. This file is the design document and the implementation plan.

Branch: `SCH-4255-align-date-time-back`

Pull request: https://github.com/calendly/booking/pull/542

Started: 2026-10-25. Pull request opened: 2026-10-25. Released: 2026-10-30.

Booking-kit in this checkout is `@calendly/booking-kit@1.0.166` (`package.json` and `yarn.lock` on `origin/main`). The last bump merged into booking is [SCH-4248](https://github.com/calendly/booking/pull/518), which pinned 1.0.166. Publish commit `0ccdc376` in `calendly/js-packages`. `js-packages` has since published 1.0.167 (`3cd56296`) and 1.0.168 (`69018c6f`). Booking `main` does not depend on either. The compare `0ccdc376...69018c6f` is 9 commits and 29 files, and none of them are under `packages/booking-kit/src/calendar/` or `packages/booking-kit/src/layout/`. The measurements below are from the 1.0.166 source and the installed CSS.

`back=1` is added in one place: `frontend/components/landing_page/event_type_card.js`, on the host landing page's event card (`eventType.url` plus `back: 1`). The booking route is `/:profileSlug/:eventTypeSlug`. `getLandingPageReferrer` in `booking_profile_back_url.js` can send the link back to a `/d/` URL. That is the return target, not how `back=1` arrives.

## Goal

On a tablet-or-wider booking page with `booking_app_redesign` on, choosing a day shows two columns. The month title is the booking-kit split header. The selected date is `DateSubtitle` in the times column. On a page opened from a host landing page, those two labels sit on one line, in the same type, and share the back link's left inset. The time slots stay aligned with the date label.

Today the back link is only in the dates column, so it pushes the month down and leaves the date at the top of the times column. The insets also differ:

- `CalendarBackRow` pads the back link by 32px on the left and right. The link itself adds `margin-bottom: 16px` (`BookingRedesignBackLink`). The control is an inline-flex row: a 16px icon and body-medium text (`font-size: var(--fontSizeBodyMedium)`, `line-height: 1.4`). The text line is about 19.6px, taller than the icon, so that is the row's content height.
- The calendar root, on tablet and up, adds `margin-left` and `margin-right` of 19px and sets `overflow: hidden`. The header adds `margin: 6px 6px 15px`. The split title has no extra left margin, so the month text starts 25px from the dates column, 7px left of the back link. The next-month button's right edge is 25px from the column (19px root plus 6px header).
- The month title is `fontSizeBodyMedium` and `fontWeightBold`, formatted `MMM yyyy` (`Jun 2026`). The nav buttons are 38×38. `data-testid="calendar-header"` is on that header in 1.0.166.
- `DateSubtitle` keeps the kit subtitle: `fontSizeTitleSmall`, `fontWeightBodyRegular`, height 38px, line-height 38px. The redesign copy is `time.formats.date_small_with_day` (`Thu, Jun 15` in English).
- `TimesHeaderRow` pads `0 ${spacing1x} 0 ${spacing0_5x}`. `spacing0_5x` is 8px and `spacing1x` is 16px. The spot list, on tablet and up, pads 8px on the left and 16px on the right. The 16px is the scrollbar gutter. SCH-4247 lined the date label up with the slots using that pair.

The day grid cannot absorb a wider root margin. Day controls are a fixed 44px circle (`margin: auto`) inside a `table-layout: fixed` table. On tablet and up the table's horizontal border-spacing is 0, so seven circles need 308px. The dates column, once a day is selected, is `flex: 1 0 350px` (it grows, and it does not shrink). The spotpicker wrapper's minimum width in that state is 585px, which is the width where the dates column sits on that 350px floor. At 350px, the current 19px margins leave 312px, about 4px more than the circles need. 32px margins would leave 286px. The circles would overlap by about 3px and the calendar's `overflow: hidden` would clip the outer edges. The circles fit again only after the row is about 657px wide. A 680px-wide picker has about 7px to spare, so a slightly narrower main panel falls short again. SCH-4270 (date-picker scaling) is Won't Do. This ticket does not change the column width.

The back row is rendered only when the flag is on, the event type is public, the query has `back=1`, the route is `/:profileSlug/:eventTypeSlug`, a profile slug is present, and the viewport is not phone. Phone is `max-width: 800px`, tablet is 801–1100px, and desktop is `min-width: 1101px` (`@calendly/ui` breakpoints). On phone the calendar and the time list are not on screen together once a day is selected, and the back control is `BookingRedesignPhoneBackButton` in the sidebar.

The Figma screenshot on the ticket (`image-20260603-143525.png`) shows the month and the date on one row, the back control on that same left edge, and the date lined up with the slot buttons. The staging screenshot (`image-20260603-143622.png`) still shows a long date (`Thursday, June 4`). That shot is from before SCH-4243. The target copy is the abbreviated redesign copy.

## Design

Keep the desktop back link where it is, in `dates.js`. Do not fork `CalendarHeader` and do not change `js-packages`. `headerVariant="split"` stays when the flag is on, and `"centered"` stays when it is off.

When that back link is actually rendered, the month title, the date label, and the time slots use its 32px left inset. The day grid stays on the kit's 19px root margin. The date label moves down so its 38px line matches the month's 38px line.

Dates column:

- `CalendarBackRow` keeps the 32px horizontal padding and the existing link. `display: flow-root` and `box-sizing: border-box` keep the link's 16px bottom margin inside the row. The row's height is the shared `--profile-back-row-height` below, so the border box is the text line plus that 16px.
- `StyledCalendar` gets a class only in that case. That class does not change the calendar root's 19px margins. It sets the header's `margin-left` to 13px through `[data-testid="calendar-header"]`. 19px plus 13px is 32px, so the month title starts on the chevron. Leave `margin-right` at 6px, `margin-top` at 6px, and `margin-bottom` at 15px. The next-month button stays 25px from the right edge of the column. The test id is not a booking-kit API. Put a comment on the selector, and check it again on every booking-kit bump. One booking class plus the attribute beats the kit header's single class.

Times column, same case only:

- `TimesHeaderRow` left padding becomes 32px. Its right padding stays `spacing1x` (16px). Its top margin is the shared `--profile-back-month-offset` below, which is the back row's height plus the header's 6px top margin, so the 38px date line starts where the 38px month line starts.
- The styled spot list's left padding becomes 32px, with a doubled selector so it beats the kit's tablet rule (two classes). The slots stay with the date label. Leave the list's 16px right padding alone.

The two lengths are CSS variables on the spotpicker row, the parent of both columns. Define them once, in `spotpicker.js`, as literal `var()` fallbacks. Do not import a JavaScript string into an Astroturf template.

```css
--profile-back-row-height: calc(
  var(--fontSizeBodyMedium, var(--fontSizeMedium)) * var(--lineHeightBodyMedium, 1.4) + 16px
);
--profile-back-month-offset: calc(var(--profile-back-row-height) + 6px);
```

The 16px is `BookingRedesignBackLink`'s `margin-bottom`. The 6px is the kit header's top margin. Both numbers live only in that one rule, with a comment that says so. `CalendarBackRow` uses `height: var(--profile-back-row-height)`. `TimesHeaderRow` uses `margin-top: var(--profile-back-month-offset)`. The variables may be present when the back link is hidden. Nothing reads them in that case.

`@calendly/ui/astroturf-theme-loader` inlines constants imported from `@calendly/ui/theme` and `@calendly/ui/spacing` only. A length imported from `booking_profile_back_url.js` is not a style export. `astroturf/loader` then fails `yarn build` with `AstroturfLoaderError: Could not find a style associated with the interpolated value`. Font tokens used for `DateSubtitle` are imported from `@calendly/ui/theme` in `times.js`, which the theme loader does inline.

Type, only when the back link is rendered:

- `DateSubtitle` uses `fontSizeBodyMedium` and `fontWeightBold`, matching the split month title, with a doubled selector so it beats the kit subtitle. Keep the 38px height and line-height.

When the redesign date row is showing and the back link is not, leave `DateSubtitle` on the kit type and leave `TimesHeaderRow` with no extra top margin. The month header still has its 6px top margin, so those two lines are about 6px apart. The ticket does not ask to move that no-back layout. Do not add a 6px offset there.

Rejected:

- Setting the calendar root's left and right margin to 32px, or zeroing the header's right margin. That was the first layout, and it clips the day circles at the dates column's 350px floor. The acceptance criteria name the month title, the date label, and the back button's inset. They do not move the day grid.
- Editing `CalendarHeader`, the month format, or any file in `js-packages`. The ticket forbids it. The split header already owns `MMM yyyy` (`Jun 2026`). The ticket's "June 2026" is the Figma picture. Rendering that full month would be a kit change. [Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269/Updating+Core+UX+Codeowner) puts `booking-kit` with Scheduling and Core UX. Staying out of that repo keeps this a Scheduling change.
- Moving the back link into `spotpicker.js` so it spans both columns. The ticket keeps the desktop link in `dates.js`. `spotpicker.js` only gains the two CSS variables on the row.
- Changing the date string to the Figma's `Thursday, Jun 15`. SCH-4243 shipped the abbreviated column copy (`date_small_with_day`). This ticket asks for the same font and size, not a new string. SCH-4243 does not change the calendar month title.
- Applying the 32px inset, or the date-label type change, when the back link is not rendered.

Flag off is unchanged: no back row, centered header (`June 2026`), kit subtitle (`Thursday, June 15`), no redesign date row.

`CalendarBackRow` and `StyledCalendar` take `isPhone` from `Dates`, which already calls `useDeviceType`. Calling that hook inside `styled().attrs` fails `react-hooks/rules-of-hooks` when ESLint lints this file, and the pre-commit hook lints the staged file. The phone padding rule stays. The row is still not rendered on phone.

## Out of scope

Sibling bugs on the same screen stay out: SCH-4256 (nav hover), SCH-4241 (mobile available-times bar), SCH-4271 (AM/PM spacing), SCH-4239 and SCH-4749 (container height), SCH-4249 (payment layout), SCH-4270 (date-picker scaling, Won't Do), SCH-4269 (slot margins, Done; same list padding, do not restyle the slots), SCH-4272 (timezone control, Product Review; same header row, do not restyle it). SCH-4247 (date label aligned with the slots) is already done. This change keeps that relationship by moving the list's left padding with the date label, and only when the back link is showing.

Also out: the phone back button, the enter-details back link, the flat spotpicker, mutual availability, polls, the no-back 6px gap between the month and the date, and the flag's rollout.

## Code changes

Files:

- `frontend/utils/booking_profile_back_url.js` — add the existing visibility check (`back=1`, public event type, route `/:profileSlug/:eventTypeSlug`, profile slug present) next to `buildProfileEventTypesListUrl`. `dates.js` and `times.js` both call it. Do not export a CSS length from this file. Leave `sidebar.js` as it is. Its copy hides the legacy sidebar back button when the redesign is on.
- `frontend/components/select_date_time/spotpicker/spotpicker.js` — set `--profile-back-row-height` and `--profile-back-month-offset` on the row that wraps `Dates` and `Times`. Do not move the back link here. `spotpicker.test.js` mocks `../dates` and `../times`. Leave that test's assertions as they are.
- `frontend/components/select_date_time/spotpicker/dates.js` — `CalendarBackRow` height and `StyledCalendar` header `margin-left`, as in Design. Pass `isPhone` into those styled components from the `useDeviceType` call already in `Dates`. Still pass `headerVariant={bookingAppRedesign ? 'split' : 'centered'}`. Do not change month navigation or day selection. Do not set a 32px margin on the calendar root.
- `frontend/components/select_date_time/spotpicker/times.js` — `TimesHeaderRow`, `DateSubtitle`, and the styled spot list, as in Design. The date format key stays `time.formats.date_small_with_day` when the flag is on. The new type rules apply only with the back link.
- `frontend/components/select_date_time/spotpicker/__tests__/dates.test.js` — new. Flag on, desktop, `?back=1`, public event, profile route: the back link is in the document, the calendar is called with `headerVariant="split"`, and the calendar carries the `profileBack` class. Flag on without `back=1`: no back link, `headerVariant="split"`, and no `profileBack` class. Flag off: no back link, `headerVariant="centered"`, no `profileBack` class. Phone: no calendar back row.
- `frontend/components/select_date_time/spotpicker/__tests__/times.test.js` — keep the abbreviated-copy assertion and the existing no-back padding assertion. Add the back-visible case: the `profileBack` class is on the date row, the spot list, and the date label, and it is absent without `back=1`. Flag off still has no redesign date row.

Jest does not check the pixel inset. `jest.config.js` sets `astroturf: true` and does not set `linaria`, so the transformer is `@calendly/ui/test/support/astroturf-jest.js`. It maps class names and does not insert a style tag. `toHaveStyle` cannot see these rules. The existing no-back `toHaveStyle({ padding })` stays because it is already on `main`. It is not proof of computed padding: an unresolved `var()` compares empty to empty. Do not add a `toHaveStyle` assertion for `32px`, the top offset, or the date-label font. Do not read the source file from the test as a stand-in for layout. The running page is that check.

No Storybook file. See Testing.

No procedure page owns this layout. The steps above are the booking column styles the ticket names. The review and rollback pages constrain what is not edited and how a shipped image is undone. Those are in Review and Rollback.

## Testing

### Local

`yarn test frontend/components/select_date_time/spotpicker/__tests__/dates.test.js frontend/components/select_date_time/spotpicker/__tests__/times.test.js`

`yarn build:dev` must compile. The extracted CSS for `CalendarBackRow` uses `var(--profile-back-row-height)`, the header rule sets `margin-left: 13px` and does not set the root margin to 32px, and `TimesHeaderRow` uses `var(--profile-back-month-offset)`.

The dates examples assert the back link, `headerVariant`, and the `profileBack` class, including its absence when `back=1` is missing and when the flag is off. The times examples assert the `profileBack` class on the date row, the date label, and the spot list when `back=1` is present, and its absence otherwise. The existing example still asserts the abbreviated date.

This repo has no Storybook. There is no `.storybook` directory, no story file, and no `storybook` script. `chromatic` is a dependency in `package.json` and has nothing to snapshot. No story is added.

Running app, before the diff is shown: clone `calendly/calendly` into this ticket folder, start it with the booking app as the [booking README](https://github.com/calendly/booking/blob/main/README.md) describes (`booking.proxy_local_url` in the monolith's `development.local.yml`, then `bin/run`, port 3002). Open a public event from a host landing page event card so the URL has `back=1`. Select a day.

Measure at 801px, 1101px, and about 1280px, with the event-details sidebar and without it (`hideEventTypeDetails`). At each of those, `getBoundingClientRect` for the back link, the month title (`[data-testid="title"]`), and the date label share a left edge and the two 38px lines share a top. The first time slot's left edge matches the date label. An outer day circle is fully inside the calendar, with no overlap into the next circle. The next-month button's right edge matches the same button on a flag-on load that has no `back=1`. Previous and next month still change the month. Choosing another day still changes the time list.

The flag-on load is the one whose `GET /api/booking/experiments` body has `enabledFeatures.booking_app_redesign` true. On that load the month reads like `Jun 2026` and the date reads like `Thu, Jun 15`. Then load with the flag off: centered full month (`June 2026`), date like `Thursday, June 15`, and no calendar-column back link. A flag-off screenshot does not verify the change. If the local experiments response has the flag false, the unit tests and `yarn build:dev` still run, and the page check waits for the shared environment. The July 2024 experimenter page says booking was not on that override tool. This clone reads the flag from `/api/booking/experiments`, so that response is the proof. Do not post an override from that page.

### Shared environment

`calendly/booking` publishes a Signadot sandbox when the pull request has the `staging-overlay-sandbox` label (`.github/workflows/sandbox-deploy.yml`). [Signadot Sandboxes Overview](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/4850876496/Signadot+Sandboxes+Overview) step "Routing Traffic": select that booking sandbox in the Signadot Chrome plugin. [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot) step 2 selects an `oap-web` MFE sandbox. That step is not this repo. Use the booking sandbox the label created.

On staging, open the same public event from a host landing page event card (`back=1`), with the plugin routing to the sandbox. Proof is the same experiments field, the same two strings on one line, and the same left-edge and day-circle checks as Local, at a desktop width. Repeat with the flag off, or on a load whose experiments field is false, and confirm the centered month and the long date are unchanged. Month navigation and day selection still work.

## Post-production

After merge, staging deploys from `main`, and production follows the `argocd-gcp` booking manifest PR described in the booking README. After that production sync, open a production booking page from a host landing page event card (`back=1`) whose `GET /api/booking/experiments` has `enabledFeatures.booking_app_redesign` true. The month (`Jun 2026`) and the date (`Thu, Jun 15`) share a line at the back link's left inset, and the day circles are not clipped. This check runs after production. It is not part of the pull request.

## Rollback

Revert the booking commit and let the normal deploy ship the previous image. That restores the previous insets and type. It does not turn `booking_app_redesign` off, and it does not change booking-kit.

If the production image is already the bad one, use [Performing Rollbacks and Reverts](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/3015344227/Performing+Rollbacks+and+Reverts), section "Gitops Deployed Apps", Basic Method, on `services/production/booking.yaml` in `argocd-gcp` (and the staging file if staging should return too). The monolith `/deploy` rollback on that page is for the release train only. There is no migration, so the page's migration warning does not apply. A rollback does not restore a visitor's in-progress selection.

## Sources

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4255. Parent epic SCH-3792. Comments from Ryan Guyer: the back link must let the two columns stay aligned, and the month and date labels must match. Screenshots on the ticket: `image-20260603-143525.png` (Figma) and `image-20260603-143622.png` (staging). The Figma shot is the layout target. The staging shot predates the abbreviated date from SCH-4243.
- [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot)
- [Signadot Sandboxes Overview](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/4850876496/Signadot+Sandboxes+Overview)
- [Performing Rollbacks and Reverts](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/3015344227/Performing+Rollbacks+and+Reverts)
- [Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269/Updating+Core+UX+Codeowner)
- [Feature flag testing with Experimenter Service](https://calendlyapp.atlassian.net/wiki/spaces/QE/pages/3233022207/Feature+flag+testing+with+Experimenter+Service) — opened because the check needs the flag. Its booking caveat is called out in Testing. It is not the override step for this plan.
- Booking README deployment and local proxy steps, in the clone.
- Searches that did not find a layout procedure: `booking_app_redesign` (hit was Core UX feature-flag governance for mf-publisher), `booking app redesign`, `booking-kit`, `Calendar header`.

## Review

None. The squad on the ticket is Scheduling.

`calendly/booking` has no `CODEOWNERS` file. Checked the clone at `origin/main` (`91ccf951`) and the GitHub tree for `.github`. The paths this change touches are the booking files in Code changes, including `spotpicker.js`. There is no later rule that assigns them to another team.

[Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269/Updating+Core+UX+Codeowner) does not list `calendly/booking`. It says Core UX is not a required reviewer for unrelated product code. It does list `booking-kit` inside `calendly/js-packages`, with Scheduling as the specialist owner. This plan does not change that repo, so Core UX is not requested. The page names no Slack channel for this change.
