---
name: SCH-5148 schedule card hover border
overview: Restore the shared schedule card so its resting border is the light standard stroke and hover uses the darker medium-hover stroke, matching the Event Type Editor v4 Figma.
branch: SCH-5148-schedule-card-hover-border
pullRequest: https://github.com/calendly/mf-publisher/pull/4178
started: 2026-10-30
pullRequestOpened: 2026-10-30
released: 2026-11-03
delivery:
  - repo: calendly/mf-publisher
    branch: SCH-5148-schedule-card-hover-border
    pullRequest: https://github.com/calendly/mf-publisher/pull/4178
issueType: Bug
storyPoints: 2
todos:
  - id: card-border
    content: Restore the schedule card resting border to colorStrokeStandard and set the clickable hover border to colorStrokeMediumHover
    status: pending
  - id: pill-rest
    content: Set the resting Edit pill border to colorStrokeStandard and keep the emphasized secondary-outline appearance
    status: pending
  - id: tests
    content: Assert the clickable card exposes data-hovered while the pointer is over it, and that locked, read-only, and custom-hours cards do not
    status: pending
  - id: stories
    content: Add Resting and Hovered Storybook stories that render those borders without a pointer
    status: pending
isProject: false
---

# Darken the schedule card border on hover

Shareable implementation note for [SCH-5148](https://calendlyapp.atlassian.net/browse/SCH-5148) (ET Scheduling · Bug · 2 SP · In Dev · P2 · Low impact).

This file is the design document and the implementation plan.

Repo: `calendly/mf-publisher`. Checkout: `/Users/somo.banerjee/Documents/projects/SCH-5148/mf-publisher` on `main`.

Branch: `SCH-5148-schedule-card-hover-border`

Pull request: https://github.com/calendly/mf-publisher/pull/4178

Started: 2026-10-30. Pull request opened: 2026-10-30. Released: 2026-11-03.

Constraint: the card already ships behind the existing single-host editor gate `sa_et_crud_single_host`. This change does not add a flag. Borders stay on `@calendly/ui` semantic tokens (`@calendly/ui` 11.0.4). No hex values in the component.

## Goal

On a shared schedule in the v4 Availability section, hovering the card makes the border lighter. The resting border is `colorStrokeMediumHover` (`#6a7884` in the light Brand 2026 theme). The clickable hover rule switches it to `colorStrokeStandardHover` (Brand 2026 light maps that token to `coreColorNeutral200`, which is lighter than `coreColorNeutral500`).

That pair was introduced in [SCH-5066](https://calendlyapp.atlassian.net/browse/SCH-5066) (`calendly/mf-publisher` pull request 4106). Before that pull request the card border was `colorStrokeStandard` (`#e4e6e9`) and there was no hover border.

The Figma frame [Schedule edit tap area](https://www.figma.com/design/4BRjJmbePNMmc8J9W6t0qg/Event-type-editor--v4-?node-id=11071-130341) draws the two states the other way around:

| State | Container border | Edit pill |
| --- | --- | --- |
| Default | `colorStrokeStandard` (`#e4e6e9`) | border `colorStrokeStandard`, surface `colorSurfaceStandard` |
| Hover | `colorStrokeMediumHover` (`#6a7884`) | border `colorStrokeMediumHover`, surface `colorSurfaceStandardHover` |

The hover frame annotation says a pointer anywhere on the schedule triggers the hover state, and to note the border color change and the edit pill color change.

The footer divider stays `colorStrokeStandard` in both frames. The code already does that.

The new editor is on when gate `sa_et_crud_single_host` is enabled, or when layer `sa_et_crud_single_host_experiment_layer` returns `show_new_ET_editor` and the product-refresh app shell is on. `?et_editor=new` forces that editor. `?et_editor=classic` forces the classic rail. The path under test is `/event_types/<id>/editor`. The visible proof is the Availability schedule card with an Edit pill in the weekly-hours header. The classic rail does not render `V4ScheduleCard`.

## Design

**Approach.** Put the resting border back on `colorStrokeStandard` for every schedule card. On the clickable shared card only, hover sets `colorStrokeMediumHover`. Drive that hover border from the same hovered state that already emphasizes the Edit pill, and also from CSS `:hover`, so a story can show the dark border without a pointer. Set the resting pill border to `colorStrokeStandard`. Leave the emphasized pill on `appearance="secondary-outline"`.

**Why this one.** It matches the Figma token pair and restores the resting border SCH-5066 replaced. Semantic tokens keep the Brand 2026 dark-theme mappings in `@calendly/ui` (`colorStrokeStandard` → `coreColorNeutral800`, `colorStrokeMediumHover` → `coreColorNeutral400`). The component does not pick hex values.

**Rejected.** Swapping the two tokens that are in the file today would put `colorStrokeStandardHover` on the resting border. That is still darker than `colorStrokeStandard`, which is the Figma default and the pre-SCH-5066 border. Leaving `colorStrokeMediumHover` as the resting border and only darkening hover further would keep the card in its hover color at rest. Changing `secondary-outline` inside `@calendly/ui` would restyle every outlined button. That appearance uses `colorStrokeStrong` and `colorSurfaceActionTertiaryHover`, not the Figma pill pair. SCH-5066 QA accepted the pill switching to the outlined style, so this ticket keeps that appearance.

**Blast radius.** Every `V4ScheduleCard` gets the light resting border again, including custom hours, locked, and read-only cards. Those cards are not clickable, so they do not gain a hover border. The clickable shared card darkens on hover and the Edit pill switches to `secondary-outline` while the card is hovered, which is the behavior that already exists. Click handling, the schedule editor, and the footer divider stay as they are.

## Out of scope

- [SCH-4696](https://calendlyapp.atlassian.net/browse/SCH-4696), the location-module hover task. It is still Ready For Dev and is a different container.
- [SCH-5130](https://calendlyapp.atlassian.net/browse/SCH-5130), the schedule editor showing the previous schedule. Unrelated to the border.
- Making custom hours, locked, or read-only cards clickable. SCH-5066 left those non-clickable, and QA confirmed that.
- Responsive layout, truncation, and the schedule-edit modal copy.
- Other `colorStrokeStandardHover` call sites (scheduling list, recaps, booking-page styles).
- A new feature flag, or a change to `sa_et_crud_single_host`.

## Code changes

Checkout: `/Users/somo.banerjee/Documents/projects/SCH-5148/mf-publisher`.

### `v4_schedule_card.js`

File: `app/src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/v4_schedule_card.js`.

1. In `scheduleCardCss`, set the base border back to `1px solid ${colorStrokeStandard}`. Remove the `colorStrokeMediumHover` import if nothing else in the file uses it. Remove the `colorStrokeStandardHover` import. The footer rule in `scheduleCardFooterCss` already uses `colorStrokeStandard` and stays.
2. On `scheduleCardClickableCss`, keep `cursor: pointer`. Apply `border-color: ${colorStrokeMediumHover}` for both `&:hover` and a hovered class that is present when the card is hovered. Reuse `isCardHovered` for that class. Add an optional prop, default `false`, that the Hovered story sets so the dark border and the emphasized pill render without a pointer. Production callers omit it. Mouse enter and leave still update `isCardHovered`.
3. Set `data-hovered="true"` on the clickable card while that hovered state is on, and `data-hovered="false"` otherwise. Non-clickable cards stay `data-clickable="false"` and do not get the hovered class.
4. Do not change `handleCardClick`, the interactive-descendant guard, or which schedules are clickable (`!isReadOnlyState && !usingCustomHours`).

### `v4_edit_availability_button.js`

File: `app/src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/v4_edit_availability_button.js`.

`restingButtonCss` currently forces `border-color: ${colorStrokeMediumHover}` while the pill is not emphasized. Change that to `colorStrokeStandard`, which is the Figma default Edit button. Keep `appearance={isEmphasized ? 'secondary-outline' : 'ghost'}` and keep clearing `className` while emphasized. Ghost in `@calendly/ui` does not set a resting border; this override is what draws the standard stroke at rest. Do not restyle `secondary-outline`.

### Storybook

[Writing Storybook Stories in mf-publisher](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/4628971555/Writing+Storybook+Stories+in+mf-publisher) says a story passes explicit props, avoids live flags and app state, and does not boot a connected editor. `mf-publisher` has Storybook (`app/.storybook`, addon `@chromatic-com/storybook`). This feature has no story today. There is no pseudo-state addon, so a CSS-only `:hover` rule would not show up in a Chromatic snapshot.

Add `app/src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/v4_schedule_card.stories.js`.

- Title under the event type editor, component `V4ScheduleCard`.
- Mock the form, `useOpenSharedScheduleEditor`, the shared-availability display, and the interval, date-specific, and timezone children the way `v4_schedule_card.test.js` does. Do not mount the CRUD editor or read `sa_et_crud_single_host`.
- **Resting.** Shared schedule (`usingCustomHours: false`), hovered prop off. The card border is the standard stroke, `data-hovered` is false, and the Edit pill is not emphasized.
- **Hovered.** Same card with the hovered prop on. The card border is `colorStrokeMediumHover`, `data-hovered` is true, and the Edit pill is `secondary-outline`. No control has to be clicked for that snapshot.
- Leave a third story out. Custom hours and locked cards are covered by the unit tests and are not a new visual state beyond the restored resting border.

Chromatic on the pull request snapshots Resting and Hovered. That is the shared-environment visual check.

## Testing

### Engineer: complete the code

From `/Users/somo.banerjee/Documents/projects/SCH-5148/mf-publisher`:

```bash
yarn --cwd app test src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/__tests__/v4_schedule_card.test.js src/javascripts/features/event_type_editor/control_panel/sections/components/availability/crud/__tests__/v4_edit_availability_button.test.js
```

Extend `v4_schedule_card.test.js`:

- Mouse enter on the clickable shared card sets `data-hovered="true"`. Mouse leave sets it back to false. The existing emphasized-pill assertion stays.
- A custom-hours card and a locked card stay `data-clickable="false"` and never `data-hovered="true"`.
- The hovered story prop is not required in these tests. They keep using mouse enter.

Open the two stories in Storybook (`yarn --cwd app storybook`) and confirm Resting is the light border and Hovered is the darker border with the outlined Edit pill, without moving the pointer. Linaria will not put the token name on the DOM, so the story is the color check. The unit test checks the hovered attribute the border class is tied to.

Chromatic on the pull request snapshots those two stories (`yarn chromatic` in `app/`, which runs `npx chromatic --only-changed`).

The engineer does not add a sandbox label, configure Signadot, or click through Alpha. No Signadot or QA handoff is required. The ticket is labeled `dev-tested`, impact is Low, and the Resting and Hovered stories distinguish the inverted tokens from the Figma pair. After those Jest tests pass, the two stories have been opened locally, Chromatic and CI pass, and a teammate has code-reviewed the pull request, the engineer is done.

## Post-production

After this `mf-publisher` change is deployed, open a single-host event type on `/event_types/<id>/editor` with `sa_et_crud_single_host` on (or `?et_editor=new`). The Availability card shows the Edit pill in the weekly-hours header. At rest the card border is the light standard stroke. Hovering the card darkens that border. The footer divider does not change. `?et_editor=classic` still opens the classic rail and does not show this card.

## Rollback

Revert the `mf-publisher` commit. That restores the inverted hover (`colorStrokeMediumHover` at rest, `colorStrokeStandardHover` on hover) and the dark resting pill border. It does not change schedule data, and there is no flag to turn off.

## Sources

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5148
- Parent epic: https://calendlyapp.atlassian.net/browse/SCH-4200
- SCH-5066, which introduced the inverted pair: https://calendlyapp.atlassian.net/browse/SCH-5066 and https://github.com/calendly/mf-publisher/pull/4106
- Figma, node `11071:130341` (Default frame `11070:106220`, Hover frame `11070:110462`): https://www.figma.com/design/4BRjJmbePNMmc8J9W6t0qg/Event-type-editor--v4-?node-id=11071-130341
- Loom on the ticket: https://www.loom.com/share/0c34143072d24aa78db18ddf08e6c06b
- Procedure opened: [Writing Storybook Stories in mf-publisher](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/4628971555/Writing+Storybook+Stories+in+mf-publisher). The stories follow its explicit-props rule and do not boot the editor.
- [Chromatic](https://calendlyapp.atlassian.net/wiki/spaces/rdops/pages/4669440281/Chromatic) is a license page, not a visual-test procedure. The repo's Chromatic check is the `chromatic` script in `app/package.json`.
- Searches that found no procedure for this token swap: Confluence `text ~ "colorStrokeMediumHover"`; titles matching hover, design tokens, or stroke token; Engineering space titles matching experiment or feature flag. This change does not add a flag, so the Optimizely feature-flag page is not a step in the diff. [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot) is not a step here because QA browser verification is not required.

## Review

None. The squad that owns the ticket is Scheduling. The last `CODEOWNERS` match for the planned files is `app/src/javascripts/features/event_type_editor/` → `@calendly/scheduling-and-availability` (the later of the two identical rules, line 142). Later rules in that file cover notetaker, settings-modal pages, and Callie activation. They do not match this path. The new story sits in that same directory, not under `app/.storybook/` (which is `@calendly/core-ux`). No outside team, channel, or review policy applies. Gate 2 checks `CODEOWNERS` again against the files actually in the diff.
