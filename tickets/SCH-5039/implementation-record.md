# SCH-5039 — Fire `editor_version` for every full-page editor share

## Metadata

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-5039
- Repo: `calendly/mf-publisher`
- Branch: not recorded
- Pull request: not recorded
- Parent: [SCH-4200](https://calendlyapp.atlassian.net/browse/SCH-4200)
- Squad / priority / points: ET Scheduling · P1 · 3 SP · High impact
- Status: Released

## Timeline

- Started: not recorded
- Pull request opened: not recorded
- Released: 2026-10-09

## Story

`Booking Page Shared` from the full-page event type editor included `editor_version: full_page_editor` only for Calendly’s internal org. Copy-link tracking returned early unless the alpha gate `sa_et_crud_single_host` was on. Experiment users, and anyone opening the editor with `?et_editor=new`, never emitted the event. The copy-link hook now uses `useEtCrudEditorAnalyticsEnabled`, the same check the other editor events already use. The event fires when the alpha gate is on, when the experiment is a treatment and the product-refresh shell is on, or when the URL has `?et_editor=new`.

Left alone: the analytics helper itself, Statsig exposure, email, embed, preview, and single-use-link actions, and the Hex dashboard. The header and the you’re-live modal already call this one hook. Their payloads stay as they are.

## Code

- `app/src/javascripts/features/event_type_editor_crud/use_track_editor_copy_link.js`: replace the direct `sa_et_crud_single_host` check with `useEtCrudEditorAnalyticsEnabled()`. Drop the unused `useNewCode` and gate imports. Leave the callback and the share-location constants.
- Extend `use_track_editor_copy_link.test.js` for gate on, experiment treatment with the shell on, treatment with the shell off, control, `?et_editor=new`, and everything off.
- Do not edit `editor_primary_actions.js` or `publish_live_modal.js`.

## Proof and rollback

On staging, as an org that is not the Calendly internal org and is in the experiment treatment with the product-refresh shell: Copy link in the header emits `Booking Page Shared` v2 with `share_location: et_editor` and `editor_version: full_page_editor`. Copy link in the you’re-live modal emits the same event with `share_location: et_editor_youre_live_modal`. Production confirmation is the Hex dashboard on the ticket: non-internal orgs show `editor_version = full_page_editor`.

The plan does not record that this check was run. It does not name a rollback.

## People

The plan does not name an outside GitHub team. Tommy called the firing paramount for measurement. Kristyn asked to pull it into the sprint. Scheduling owns the event type editor.

## Still open

Branch and pull request are not in the plan. The production dashboard check is not recorded.

Plan: tickets/SCH-5039/sch_5039_implementation_plan.plan.md
