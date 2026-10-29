---
name: SCH-4908 overflow submenu keyboard
overview: Page language and Event type color in the new editor overflow menu become CUI submenus, so the keyboard can open them and the parent row stays highlighted while open.
branch: SCH-4908-overflow-submenu-keyboard
pullRequest: https://github.com/calendly/mf-publisher/pull/4121
started: 2026-10-24
pullRequestOpened: 2026-10-24
released: 2026-10-29
delivery:
  - repo: calendly/mf-publisher
    branch: SCH-4908-overflow-submenu-keyboard
    pullRequest: https://github.com/calendly/mf-publisher/pull/4121
todos:
  - id: submenu-rows
    content: Render Page language and Event type color as DropdownSubmenu items and delete the hover flyout, stamp helper, and extra portal host
    status: pending
  - id: open-highlight
    content: Paint the parent row with the menu hover background while aria-expanded is true
    status: pending
  - id: keyboard-tests
    content: Cover ArrowRight open, focus return, parent open state, and the existing color and locale commits
    status: pending
  - id: story
    content: Add a Storybook story that snapshots each submenu already open
    status: pending
isProject: false
---

# Keyboard access for overflow submenus

Design document and implementation plan for [SCH-4908](https://calendlyapp.atlassian.net/browse/SCH-4908).

Repo: `calendly/mf-publisher`. Default branch is `main`. Checkout: `/Users/somo.banerjee/Documents/projects/SCH-4908/mf-publisher`. `@calendly/ui` stays at 11.0.4 (`app/package.json` resolutions). No feature flag is added. The new editor is already gated by `sa_et_crud_single_host`, the `sa_et_crud_single_host_experiment` treatment, or `?et_editor=new`.

Branch: `SCH-4908-overflow-submenu-keyboard`

Pull request: https://github.com/calendly/mf-publisher/pull/4121

Started: 2026-10-24. Pull request opened: 2026-10-24. Released: 2026-10-29.

## Goal

On the new event type editor, keyboard users cannot open the secondary menus inside the header overflow menu, and the parent row loses its highlight once a secondary menu is open.

`HeaderMoreMenu` renders Page language and Event type color as `DropdownMenuItem` rows with `disableAutoClose`. Inside each row, `PageLanguageMenu` and `EventColorMenu` mount a second `DropdownMenu`, opened by `BareButton` plus `useHoverOpenMenu`. Hover works because `useHoverOpenMenu` listens for pointer movement. The keyboard does not: arrow keys and Enter stay on the React Aria `menuitem`, that item has no `onSelect`, and `disableAutoClose` swallows the activation. The nested button never receives the key.

The highlight fails for the same split. The parent row's hover background is the `menuitem` `:hover` / `:focus` style. The flyout is a second portal, so moving into it clears both. Nothing sets an open style on the parent row.

Both rows are in the kebab used by `editor_header.js` and `editor_primary_actions.js`. Page language stages `locale` on the editor form after the existing confirmation popup. Event type color stages `color` on that form. Neither row saves on its own.

## Design

Use CUI `DropdownSubmenu` for those two rows. In `@calendly/ui` 11.0.4, `DropdownSubmenu` wraps a `DropdownMenuItem` that has `submenu` and `submenuKey` in React Aria `SubmenuTrigger` on desktop, and on phone it leaves the item in place so `DropdownMenuItem` pushes the submenu onto the menu stack. That is the pattern in the Storybook example the ticket links (`navigation-dropdownmenu--with-icons-and-submenus`).

Desktop keyboard and hover then come from `SubmenuTrigger`: ArrowRight, Enter, and hover open the submenu, focus moves into it, ArrowLeft and Escape close it and return focus to the parent row. `submenuKey` already tells `DropdownMenuItem` not to close the parent menu on the row's own activation, and to render the chevron. A chosen locale or color is still an action item whose `onSelect` runs and whose own close closes the kebab. The confirmation popup and the form writes stay in `HeaderMoreMenu`.

The outside-click race that the current comments give as the reason for a second menu is handled in this version of CUI. `DropdownSubmenuContent` stamps the submenu popover with the parent menu instance id (`dropdownInstanceProps`). The package test "keeps desktop submenu clicks in the shared layer host from closing the menu" covers a click in that submenu. `stampFlyoutWithParentInstance` was the product copy of that stamp. A search of the 11.0.4 dropdown-menu source, its metadata, and current `master` did not find a warning that submenus cannot hold a selection list. These lists are action items with a check icon. They do not use `selectionMode`.

Phone width follows CUI `useDeviceType('isPhone')`, which is the branch inside `DropdownSubmenu`. Enter on the row stacks the locale or color list in the same sheet, with the existing back control. The editor's `isPhone` prop stays what it is today: kebab appearance and menu width. It is a different check, and this change does not add another breakpoint.

The open highlight is the remaining gap in CUI. React Aria sets `data-open` on the parent `menuitem` while `aria-expanded` is true (`MenuItem` in react-aria-components 1.6.0). `DropdownMenuItem`'s `baseContainer` paints `:hover`, `:focus`, and `:focus-within` only, on 11.0.4 and on current `master`. Focus moves into the portaled submenu, so those selectors go idle. `DropdownMenuItem` also overwrites `className`, so the row cannot take a class from the editor. The kebab `DropdownMenuContent` gets a class that sets `background` to `colorSurfaceActionSecondaryHover` on `[data-open="true"]`. That is the same token CUI uses for the hover and focus background. The selector is limited to this menu.

Rejected:

- Keep the second menu and add keyboard handlers to `useHoverOpenMenu`. The keydown target is the `menuitem`, not the nested `BareButton`. A parallel keyboard machine would keep the hover hook, the stamp helper, and the extra portal host, and it would still miss the open style CUI expresses with `data-open`.
- Fix the highlight in `calendly/ui`. mf-publisher resolves `@calendly/ui` to 11.0.4, and current `master` still has no `[data-open]` rule. A package change would not reach this editor in this change.
- Render the locale and color lists with `selectionMode="single"`. That path draws CUI's radio or checkbox chrome. The check icon stays.

Blast radius is the two submenu rows and the helpers only they use. `useHoverOpenMenu` and `stampFlyoutWithParentInstance` have no other callers. The question-type menu is a standalone dropdown on the booking-page tab. It stays as it is. The on/off switch, Rename, Duplicate, Delete, and the other kebab actions stay as they are.

## Out of scope

- [SCH-4734](https://calendlyapp.atlassian.net/browse/SCH-4734) and [SCH-4895](https://calendlyapp.atlassian.net/browse/SCH-4895). They are other menu surfaces.
- `question_type_menu.js`. The color-menu comment points at it as a shape reference. It is not a submenu of this kebab.
- A `@calendly/ui` version bump, and any edit under `calendly/ui`.
- Multi-host editor work gated only by `sa_et_crud_multi_host`, except where that shell already renders this same `HeaderMoreMenu`.
- Changing when View live page is omitted at the large-desktop breakpoint.

## Code changes

Checkout: `/Users/somo.banerjee/Documents/projects/SCH-4908/mf-publisher`.

Manual Accessibility Checklist, Keyboard, Navigation: Enter/Space performs the control's action, and arrow keys move inside a menu. Keyboard, Focus: the focused control has a visible indicator, and focus moves to the next logical element after the action. Closing returns focus to the element that opened it. Those steps are met by `SubmenuTrigger` plus the open background below.

1. `page_language_menu.js`. Export a `DropdownSubmenu` whose only child is a `DropdownMenuItem` with `submenuKey="page-language"`, `data-testid="header-more-menu-page-language"`, the language icon, and the existing label. `submenu` is the locale `DropdownMenuItem` list, still using `header-more-menu-locale-${identifier}` and the check icon for `activeLocale`. `onSelect` calls the existing `onSelectLocale`. Delete the inner `DropdownMenu`, `BareButton`, manual chevron, `useHoverOpenMenu`, and `stampFlyoutWithParentInstance`. `menuPortalRoot` and `onLocaleSelected` go away. The parent item's `submenuKey` is what keeps the kebab open until a locale item runs.
2. `event_color_menu.js`. Same shape, `submenuKey="event-color"`, `data-testid="header-more-menu-event-color"`, the swatch icon, and the color items unchanged (`header-more-menu-color-${i18nKey}`, check against `activeColor`). `onSelect` calls `onSelectColor`.
3. `header_more_menu.js`. Drop `disableAutoClose` on the two rows. Keep `handleSelectLocale`, `handleConfirmLocale`, `setColorValue`, and both popups. Remove the `menuPortalRoot` host effect and stop passing `menuPortalRoot`, `onLocaleSelected`, and `onColorSelected`. Child `onSelect` closes through the parent menu's `onClose`, which is the kebab's `setIsMenuOpen(false)`. Add the open-row class on this `DropdownMenuContent`:

   `colorSurfaceActionSecondaryHover` on `& [data-open="true"]`.

   `DropdownMenuItem` does not accept a class, so the attribute selector is the hook React Aria already sets. If `local/no-bare-attributes` flags it, disable that rule on this block with that reason.
4. Delete `hooks/use_hover_open_menu.js`, `hooks/__tests__/use_hover_open_menu.test.js`, and `components/stamp_flyout_parent_instance.js`.
5. `components/__stories__/header_more_menu.stories.js`. Title `Event Type Edit/Header more menu` (`Event Type Edit` is an allowed Storybook root). Render `HeaderMoreMenu` inside `EventTypeProvider` (event type id present, so `isCreation` is false) and `Form` initial values `{ color, locale }` matching the unit-test fixture. The preview already supplies the Redux store and `MemoryRouter`. Two stories, `LanguageSubmenuOpen` and `ColorSubmenuOpen`. Each `play` opens the kebab and hovers that row, then expects the submenu item and `aria-expanded="true"` on the parent row. Chromatic snapshots after `play`, same as `folder_menu.stories.tsx`. The captured frame is the open submenu. Light is the default canvas (`#FFFFFF` in `app/.storybook/preview.js`). No dark-mode story: the change is not a dark-mode change. `colorSurfaceActionSecondaryHover` already follows the theme, and the dark-mode addon stays available.
6. `header_more_menu.test.js`. Point the color and language helpers at the `menuitem` test ids. Hover with `userEvent.hover`. Add keyboard coverage for both rows: open the kebab, move to the row, ArrowRight opens the submenu and sets `aria-expanded="true"` and `data-open="true"` on that row, ArrowLeft closes the submenu and returns focus to the row, Enter on a new locale opens "Update page language?" without dirtying the form, Enter on a color dirties the form and does not call `updateEventType`. Keep the existing confirmation, cancel, active-locale, and real-click regression cases. The portal-stamp example asserts the submenu option is not under `[inert]` and that the submenu popover carries the parent `data-cui-dropdown-instance`. The custom host assertion goes away with the host.

What stays: the on/off switch and its `disableAutoClose`, every other kebab item, the large-desktop View live page omission, draft rows staying disabled, the confirmation copy, and the form field names `color` and `locale`.

## Testing

### Local

From `mf-publisher/app`:

```bash
yarn test src/javascripts/features/event_type_editor_crud/components/__tests__/header_more_menu.test.js
yarn lint:storybook-structure
```

The new examples assert ArrowRight opens each submenu, `aria-expanded` and `data-open` on the parent row, ArrowLeft returns focus to that row, locale confirmation still gates the form write, and a color selection dirties `color` without its own PATCH.

Storybook: `yarn storybook` from `app/`. Open `Event Type Edit/Header more menu` and confirm `LanguageSubmenuOpen` and `ColorSubmenuOpen` finish their `play` functions with the submenu visible and the parent row on the hover background. Light is the default canvas.

Running app, before the pull request: the per-session workflow in [Local Development Guide for Frontend Changes in mf-publisher Using Signadot and Marina](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6016827403/Local+Development+Guide+for+Frontend+Changes+in+mf-publisher+Using+Signadot+and+Marina). Start mf-publisher, open the MFE Signadot sandbox, turn on the local branch, and load a single-host event type on the new editor. Proof you are on that editor: the address contains `/event_types/{id}/editor`, and the header shows the kebab (`editor-header-kebab`) with the Meeting details tab. If `sa_et_crud_single_host` is off, add `?et_editor=new` (see `routes.test.js`). Then, keyboard only: open the kebab, ArrowRight on Page language and on Event type color, confirm the parent row keeps the hover background while the list is open, ArrowLeft returns to the row, Enter on a locale opens the confirmation, Enter on a color enables Save changes. Repeat Enter on the row at a phone width and confirm the list stacks in the sheet with the back control. The alternate on that same page (local monolith plus local mf-publisher) is the fallback if Marina is unavailable. The flag-off classic editor is the unchanged path: it does not render this kebab.

### Shared environment

After the pull request exists. mf-publisher UI on staging uses the MFE sandbox, not a new sandbox deploy. [Testing UI Changes with Signadot MFEs](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/4958847019/Testing+UI+Changes+with+Signadot+MFEs) and [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot):

1. Open the Signadot extension, select the MFE sandbox, and confirm the staging banner.
2. On the MFE Config page, set the app to `mf-publisher` and the branch to this PR branch, then Update Project Paths. The PR build has to be green first. Signadot rewrites `/` in the branch name to `-`.
3. Open `/event_types/{id}/editor` on staging. The new-editor proof is the same as local: that path, the kebab, and the Meeting details tab. Use `?et_editor=new` when the flag is off.
4. Keyboard through Page language and Event type color as in the local steps. Proof is the open list, the parent row's hover background, and focus back on the row after ArrowLeft. Enter on a color leaves Save changes available. Enter on a new locale shows "Update page language?".
5. On a phone-width viewport, Enter on each row stacks the list and the back control returns to the kebab items.
6. Rename still runs on Enter and does not open a second list.

Chromatic is the visual check for the story. The workflow runs when the PR touches `app/src/javascripts/**` and the `run-chromatic` label is present (`.github/workflows/chromatic.yml`). Add that label. The snapshot is `LanguageSubmenuOpen` and `ColorSubmenuOpen` after `play`.

## Post-production

After the merge to `main` publishes mf-publisher. In production, open `/event_types/{id}/editor` on an account that gets the new editor (flag, experiment treatment, or `?et_editor=new`). Repeat the keyboard proof: ArrowRight opens Page language, the parent row keeps the hover background, ArrowLeft returns focus, Enter on a new locale shows the confirmation. Same for Event type color and Save changes. Confirm the running SHA with `DD_RUM.getInitConfiguration()?.version` (Rollback and Debugging Guide, "In the Browser Console") and match it to the merge commit. This check waits until production.

## Rollback

Ordinary undo is Option 2 on [Rollback and Debugging Guide for Publisher Frontend](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/4652236830/Rollback+and+Debugging+Guide+for+Publisher+Frontend): GitHub Revert, merge to `main`, let CI publish. That restores the hover-only flyouts.

If production has to leave the bad SHA before that build finishes, Option 1 runs the GCS Rollback workflow (`gcs-rollback.yml`) with the last good SHA and `environment: production`. Option 1 only repoints `manifest.json`. It leaves the commit on `main` until Option 2 merges.

Neither option clears a color or locale the user already saved with Save changes. Neither option restores keyboard access once the old assets are served again. There is no flag to turn this fix off by itself.

## Sources

- [SCH-4908](https://calendlyapp.atlassian.net/browse/SCH-4908), including the triage comment on the ticket and the Slack source linked there: https://calendly.slack.com/archives/C0B6XRD8684/p1789480700970909
- CUI reference the ticket names: https://calendly.github.io/ui/?path=/story/navigation-dropdownmenu--with-icons-and-submenus
- [Manual Accessibility Checklist](https://calendlyapp.atlassian.net/wiki/spaces/QE/pages/2509439331/Manual+Accessibility+Checklist), Keyboard, Navigation and Focus
- [Accessibility issues fixing guideline](https://calendlyapp.atlassian.net/wiki/spaces/EN/pages/2274656331/Accessibility+issues+fixing+guideline). No keyboard-menu section. The color-contrast section says to consult the squad designer for a squad-specific color. This highlight uses the existing CUI hover token.
- [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot)
- [Testing UI Changes with Signadot MFEs](https://calendlyapp.atlassian.net/wiki/spaces/PLATFORM/pages/4958847019/Testing+UI+Changes+with+Signadot+MFEs)
- [Local Development Guide for Frontend Changes in mf-publisher Using Signadot and Marina](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6016827403/Local+Development+Guide+for+Frontend+Changes+in+mf-publisher+Using+Signadot+and+Marina)
- [Rollback and Debugging Guide for Publisher Frontend](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/4652236830/Rollback+and+Debugging+Guide+for+Publisher+Frontend), Option 1 and Option 2
- [Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269/Updating+Core+UX+Codeowner)
- Confluence searches with no procedure page: `DropdownSubmenu`; titles `DropdownMenu`, `CUI`, and `Calendly UI` with text `submenu`; titles `WAI-ARIA menu`, `menu button`, and `keyboard interaction`.

## Review

No outside team.

`CODEOWNERS` last match for every file this change touches is `app/src/javascripts/features/event_type_editor_crud/` → `@calendly/scheduling-and-availability`. That is the Scheduling squad on SCH-4908.

Core UX (`@calendly/core-ux`) owns `.github/`, Storybook tooling, and CUI. [Updating Core UX Codeowner](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6252298269/Updating+Core+UX+Codeowner) requests that team for Brand Refresh, CUI, and shared frontend infrastructure. This change does not edit those paths or `calendly/ui`. The open background is local because `DropdownMenuItem` does not style `data-open` and does not accept a class.

The accessibility guideline's designer check is for a new squad-specific color. The row uses `colorSurfaceActionSecondaryHover`. No design review.

No Slack channel is named for this change.
