---
name: SCH-4740 font token size
overview: On the new event type editor Confirmations tab, raise "After booking…" and the schedule-another card title from the 12px label token to the 14px label token the rest of this editor uses.
branch: SCH-4740-confirmation-font-token
pullRequest: https://github.com/calendly/mf-publisher/pull/4123
started: 2026-10-25
pullRequestOpened: 2026-10-25
released: 2026-10-30
delivery:
  - repo: calendly/mf-publisher
    branch: SCH-4740-confirmation-font-token
    pullRequest: https://github.com/calendly/mf-publisher/pull/4123
todos:
  - id: size-after-booking
    content: Set the After booking LabelText to size large on the editable and locked Confirmations tab
    status: pending
  - id: size-schedule-another
    content: Set both schedule-another card titles to LabelText size large
    status: pending
  - id: story
    content: Add a Storybook story that renders those two strings at the new size with no extra click
    status: pending
  - id: verify
    content: Run the confirmation tab Jest test, open the story, and check the flag-on editor
    status: pending
isProject: false
---

# SCH-4740 — Confirmations tab font token

[SCH-4740](https://calendlyapp.atlassian.net/browse/SCH-4740) in `calendly/mf-publisher`. Checkout is `/Users/somo.banerjee/Documents/projects/SCH-4740/mf-publisher` on `main`. The new editor is behind `sa_et_crud_single_host` (`EVENT_TYPE_EDITOR_CRUD_GATE`). This file is the design document and the implementation plan.

Branch: `SCH-4740-confirmation-font-token`

Pull request: https://github.com/calendly/mf-publisher/pull/4123

Started: 2026-10-25. Pull request opened: 2026-10-25. Released: 2026-10-30.

The screenshot on the ticket annotates **After booking…**. Bill Siggelkow’s comment names the **Schedule another event** card, and Ryan Guyer agreed the smaller token was a design mistake. The card title is whatever is stored in `anotherEventButtonLabel`. The product default is "Schedule another event". The i18n fallback, used when that field is empty, is "Schedule another meeting".

## Goal

On the Confirmations tab, two strings render at 12px while the same kind of text elsewhere in this editor renders at 14px.

`LabelText` defaults to `size="medium"`. Every theme, including the default, uses the Brand 2026 type scale (`calendly/ui` `src/themes/README.md`, spreading `brand2026Typography`). On that scale `--fontSizeLabelMedium` is `0.75rem` (12px) and `--fontSizeLabelLarge` is `0.875rem` (14px). Weight stays `--fontWeightLabel` (500).

- **After booking…** is `LabelText` with no size in `confirmation_tab.js`, so it is 12px. The red arrow on the ticket screenshot points at this label. Section labels on Meeting details and Questions use `LabelText size="large"` (14px).
- **Schedule another event** (the stored label, or "Schedule another meeting" when the field is empty) is `LabelText` with no size in `schedule_another_meeting_card.js`, collapsed and expanded. Collapsed card titles on questions, locations, and scheduling limits use `LabelText size="large"`.

The description under the card, "Directs the invitee to schedule another meeting", is already `BodyText size="small"` (`--fontSizeBodySmall`, also `0.75rem`). That matches the secondary line on location rows and the custom-link URL in the same list. It stays.

## Design

Pass `size="large"` to the two labels that are one step below the editor’s pattern. Do not change the type scale, and do not set a raw `font-size`.

Why this one: the tokens are already correct. This screen selected the smaller `LabelText` step. `size="large"` is the step Meeting details, Questions, custom-question cards, location rows, and scheduling-limit rows already use.

Rejected:

- Editing `--fontSizeLabelMedium` in `@calendly/ui`. That would shrink or grow every medium label in the product.
- Raising the card description to `BodyText size="medium"`. Secondary lines on this editor stay small.
- Raising the custom-link titles in the same list, the Additional links subtitle, the redirect URL label, or the payments "require payment" card. They use the same unsized `LabelText`, and the ticket does not name them.
- A raw pixel size. The styling page’s examples do that. This file already uses CUI `LabelText`, and the CUI page says to use those components.

Blast radius is the Confirmations tab of the full-page editor, flag on. Copy, form values, the classic editor, and the invitee-facing confirmation page do not change.

## Out of scope

- Custom link cards in `custom_link_card.js`, the Additional links subtitle in `internal_section.js`, and the redirect URL label in `external_section.js`.
- `require_payment_card.js` on the Payments tab, which uses the same unsized `LabelText` plus `BodyText size="small"`.
- The classic editor under `features/event_type_editor/control_panel/sections/components/confirmation_page/`.
- The invitee-facing "Schedule another event" button on the booking confirmation page.
- Nearby Confirmations work: SCH-4690 (confirmation preview), SCH-4773 (helper text), SCH-4683 (Notifications tab label size).
- Copy, validation, the enabled switch, and the `sa_et_crud_single_host` gate.

## Code changes

Follow the CUI page: use the existing typography component rather than a new style. `LabelText` already has `size="large"`. No `#guild-ui` ask, because that channel is for a design that cannot be built with existing CUI.

1. `app/src/javascripts/features/event_type_editor_crud/tabs/confirmation/confirmation_tab.js`
   - On both the editable and locked tabs, set `size="large"` on the `LabelText` that renders `publisher.event_type_editor_crud.confirmation.after_booking_label` ("After booking…").
   - Leave the page title (`TitleText level={2}`) and the subtitle (`BodyText size="large"`) as they are.

2. `app/src/javascripts/features/event_type_editor_crud/tabs/confirmation/schedule_another_meeting_card.js`
   - Set `size="large"` on both `LabelText` nodes: the expanded header title and the collapsed title (`displayLabel`).
   - Leave `BodyText color="muted" size="small"` on the description.

3. `app/src/javascripts/features/event_type_editor_crud/tabs/confirmation/__stories__/confirmation_tab.stories.js`
   - New story. Title root must be `Event Type Edit` so `app/scripts/lint-storybook-structure.mjs` accepts it. Use `Event Type Edit/Confirmations/Confirmation tab`.
   - Render `ConfirmationTab` inside `Form` with `confirmationPageType: 'internal'`, `anotherEventButtonEnabled: true`, `anotherEventButtonLabel: 'Schedule another event'`, `customLinks: []`, `redirectConfigurationUrl: ''`, and `redirectConfiguration: { passBookingDetails: false }`. `InternalSection` calls `customLinks.filter(...)`, so `customLinks` is required. Storybook already provides Redux, the router, and `useBodyStyles`.
   - The story shows "After booking…" and the card title "Schedule another event" as soon as it loads. No control has to be clicked for the Chromatic snapshot. This change is not about dark mode, so there is no separate dark story. The Theme toolbar can still switch themes; the font-size tokens are the same on every theme.

The existing Jest file stays. It checks behavior and copy, and Linaria does not put the size prop on the DOM, so it cannot assert the computed font size.

## Testing

### Local

From `mf-publisher/app`:

```bash
yarn test src/javascripts/features/event_type_editor_crud/tabs/confirmation/__tests__/confirmation_tab.test.js --watchAll=false
```

That run asserts the tab still shows "After booking…", the schedule-another card, and the edit/toggle behavior. It does not assert pixels.

Open the story with `yarn storybook` and load `Event Type Edit/Confirmations/Confirmation tab`. Confirm "After booking…" and "Schedule another event" compute to `font-size: 0.875rem` and `font-weight: 500`. Confirm the description under the card is still `0.75rem`.

Running app, before the pull request, using the mf-publisher Signadot + Marina guide (frontend-only; the monolith pod stays vanilla):

1. Start `mf-publisher` (`docker compose up -d` from the clone) and confirm `https://mf-publisher.local.calendly-internal.com/manifest.json` loads.
2. Open the `mfe` Signadot sandbox, set the `mf-publisher` override to `local`, and reload.
3. Open `/event_types/:id/editor/confirmation` for a one-on-one event.

Flag-on proof is the full-page editor itself: Confirmations in the left nav and the booking preview beside it. That shell is `sa_et_crud_single_host` (or a non-control group of `sa_et_crud_single_host_experiment` while the product-refresh shell is on). On that page, "After booking…" and the schedule-another title are `0.875rem`. The description stays `0.75rem`. The classic editor (flag off, or experiment group `control`) does not render `ConfirmationTab`; its confirmation section is unchanged.

### Shared environment

After the pull request exists, and before review:

- Add the `staging-overlay-sandbox` label. `.github/workflows/sandbox-deploy.yml` publishes the sandbox only when that label is present. Route staging with the Signadot extension ([How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot)). Open `/event_types/:id/editor/confirmation`. Proof the flag is on: the full-page Confirmations tab and preview pane. Proof of this change: computed `font-size: 0.875rem` on "After booking…" and on the schedule-another title. Open a flag-off account and confirm the classic confirmation section is unchanged.
- Add the `run-chromatic` label. `.github/workflows/chromatic.yml` starts on `app/src/javascripts/**` changes and skips the snapshot unless that label is present. Chromatic snapshots `Event Type Edit/Confirmations/Confirmation tab` with both strings visible and no control to click.

## Post-production

After this branch is merged to `main` and the `mf-publisher` assets are deployed, open `/event_types/:id/editor/confirmation` on an account where `sa_et_crud_single_host` is on. "After booking…" and the schedule-another title compute to `0.875rem`. The description under the card stays `0.75rem`. This check does not run before that deploy.

## Rollback

Revert the pull request and wait for the next `mf-publisher` asset deploy. That restores the 12px label token on these two strings. It does not change saved event types, the feature gate, or the invitee-facing confirmation page.

## Sources

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4740
- Screenshot on the ticket, saved in this folder as `image-20260817-160433.png`. Arrow on "After booking…".
- Comments: Ryan Guyer (2026-08-17) "Double check font token size used here. Looks too small." Bill Siggelkow (2026-08-18) "The font seems kind of small on the Schedule another event section." Ryan Guyer (2026-08-18) "Agreed. This was probably on me accidentally selecting a smaller than necessary font token in design."
- [3. Calendly UI (CUI). Design library and components.](https://calendlyapp.atlassian.net/wiki/spaces/EN/pages/3247800327/3.+Calendly+UI+CUI+.+Design+library+and+components.) Use CUI typography. Ask `#guild-ui` only when the design cannot be built with existing CUI.
- [4. Making thing pretty. Styling components.](https://calendlyapp.atlassian.net/wiki/spaces/EN/pages/3247472797/4.+Making+thing+pretty.+Styling+components.) Read. Its examples set raw `font-size` with Astroturf. This editor already uses Linaria and `LabelText`. The plan follows the CUI page and does not copy the raw pixel example.
- Brand 2026 type scale, in the dependency rather than Confluence: `calendly/ui` `src/themes/README.md` and `src/themes/brand-2026-typography-overrides.js`. Figma reference named there: [CUI typography](https://www.figma.com/design/3a6u1VGLSyy76ZwL0OtUwI/%F0%9F%9A%A7-CUI-%7C-Calendly-UI?node-id=30052-63266).
- Confirmations tab design history, not the size spec: [SCH-4213](https://calendlyapp.atlassian.net/browse/SCH-4213).
- [Local Development Guide for Frontend Changes in mf-publisher Using Signadot and Marina](https://calendlyapp.atlassian.net/wiki/spaces/UX/pages/6016827403/Local+Development+Guide+for+Frontend+Changes+in+mf-publisher+Using+Signadot+and+Marina)
- [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot)
- Searches that did not find a second typography procedure: Confluence `title ~ "typography"`, `text ~ "font token" AND text ~ "typography"`, `title ~ "design review" AND text ~ "pull request"`, space CUI `title ~ "typography"`.

## Review

None outside the Scheduling squad.

`CODEOWNERS` last match for `app/src/javascripts/features/event_type_editor_crud/` is `@calendly/scheduling-and-availability`. That is the squad on the ticket. A story under that directory stays with the same owner. `app/.storybook/` is `@calendly/core-ux`; this change does not edit that directory.

The CUI page sends `#guild-ui` a design that cannot be built with existing components. `LabelText size="large"` already exists, so that ask does not apply. The styling page points styling questions at `#guild-frontend`. It does not require that channel to approve a size prop on a component the squad owns.

Ryan Guyer’s comment is the design input this plan implements. It does not add a second design review.
