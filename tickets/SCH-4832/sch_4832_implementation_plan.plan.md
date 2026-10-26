---
name: SCH-4832 pre-publish ET URL
overview: In the new editor, before publish, the URL mirrors the meeting name, the user can edit that slug, and leaving the name field asks the existing slug endpoint for the next free slug. The meeting name stays what the user typed.
branch: SCH-4832-pre-publish-et-url
pullRequest: https://github.com/calendly/mf-publisher/pull/4103
started: 2026-10-21
pullRequestOpened: 2026-10-21
released: 2026-10-23
todos:
  - id: mirror-and-edit
    content: On a draft, keep the live URL preview, add the same Edit control the published event type already has, and never write the slug back into the name
    status: pending
  - id: unique-on-blur
    content: On meeting-name blur, when the slug has not been customized, GET slugs/new and show that slug, including in an open Edit field. Clear a stale slug only while Edit is closed
    status: pending
  - id: send-slug
    content: On create, reject a customized slug that fails the existing blank, format, or reserved checks. Otherwise send the customized slug, or the slug slugs/new returns for the current name
    status: pending
  - id: slug-tests
    content: Assert the draft preview, the blur request, a failed check, the create gate, a customized slug that name changes do not overwrite, and the published editor left as it is
    status: pending
isProject: false
---

# Let a draft event type URL be edited before publish

Shareable implementation note for [SCH-4832](https://calendlyapp.atlassian.net/browse/SCH-4832) (ET Scheduling · 2 SP · Ready For Dev · P2).

SLA Breach Date is not set. Parent: [SCH-5099](https://calendlyapp.atlassian.net/browse/SCH-5099).

Repo: `calendly/mf-publisher`. Local checkout: `/Users/somo.banerjee/Documents/projects/SCH-4832/mf-publisher`, a fresh clone of `main` at `0527cd8ca9`.

Branch: `SCH-4832-pre-publish-et-url`

Pull request: https://github.com/calendly/mf-publisher/pull/4103

Started: 2026-10-21

Pull request opened: 2026-10-21

Released: 2026-10-23

Constraint: `sa_et_crud_single_host` (or `?et_editor=new`). This is the v4 CRUD editor. Pre-publish is `isCreation` (`eventType.id` is null). The slug check is the existing `GET /api/users/:user_id/event_types/slugs/new`.

## Goal

A new one-on-one opens with an empty meeting name ([SCH-4761](https://calendlyapp.atlassian.net/browse/SCH-4761) removed the "New Meeting" seed). Under that field, a draft only shows a preview. There is no Edit control until the event type has been published.

```261:263:mf-publisher/app/src/javascripts/features/event_type_editor_crud/tabs/meeting_details/meeting_name_field.js
  if (isCreation) {
    return <LiveUrlPreview profileSlug={profileSlug} nameValue={nameValue} />
  }
```

`LiveUrlPreview` already rewrites the visible slug from the name as the user types (`Coffee Chat!` becomes `coffee-chat`). It does not write that slug onto the form, and it does not check whether the slug is free. The create POST sends `name` and no `slug`. `EventTypes::SlugService.generate` then fills a nil slug at publish, using `Slugs::GeneratorService` against the profile's event types and `EventType::RESERVED_SLUGS`. The first collision is `coffee-chat-1`. `coffee-chat-2` is what that generator returns when `coffee-chat-1` is taken too. The ticket's `coffee-chat-2` example is that generator, not a rule that every collision skips `-1`.

The same generator is already exposed for a preview. Logged-in `GET /api/users/me/event_types/slugs/new?name=My%20Meeting` returns `{ "slug": "my-meeting-1" }` when `my-meeting` exists (`spec/requests/api/internal/event_types/slug_spec.rb`). Team drafts use `GET /api/pages/:page_id/event_types/slugs/new`. `basePath(eventType)` already builds the matching `/event_types` prefix. mf-publisher does not call this endpoint today.

After publish, Edit opens `SlugEditField`, and `handleSlugBlur` checks blank, `SLUG_REGEX`, and `RESERVED_SLUGS`. That path stays.

Interpretation this plan implements, from the user story plus the acceptance criteria:

- While the name is typed, and the slug has not been customized, the URL under the field mirrors the name. Edit closed, that is the preview. Edit open, that is the slug input.
- Edit, the same control a published event type already has, is available before publish. A slug the user types is the one that is saved. Opening Edit does not by itself count as typing one.
- Leaving the meeting name field, when the slug has not been customized, loads the slug from `slugs/new` and shows that value, suffix included. This still happens when Edit is open. The response is what the slug input shows, unless the user has typed in that input before the response arrives.
- Nothing in that update writes the slug, the suffix, or "New Meeting" into the meeting name. [SCH-4761](https://calendlyapp.atlassian.net/browse/SCH-4761) stays intact.

[SCH-4909](https://calendlyapp.atlassian.net/browse/SCH-4909) asks for this same behavior: edit the URL before publish, check uniqueness on blur, and append the next free number. This ticket is that implementation. Close SCH-4909 as a duplicate. Do not build it again.

## Out of scope

- [SCH-5065](https://calendlyapp.atlassian.net/browse/SCH-5065). Won't Do, already marked as a duplicate.
- The published editor. `CommittedUrlPreview`, Edit, and `handleSlugBlur` stay as they are. Name blur does not call `slugs/new` once `isCreation` is false.
- The classic editor (`sa_et_crud_single_host` off, and no `et_editor=new`). Drafts there still omit the slug and the server still derives it (`booking_link/utils.js`).
- `editor_header_title.js`. An empty name still displays "New Meeting" in the header. Renaming from the header shares the `name` form value, so the preview updates, and this ticket does not add the uniqueness request to that header blur. Publish still asks `slugs/new` for an uncustomized name, so a header rename is not dropped on create. The suffix on the Meeting details line still appears when that field blurs.
- `calendly/calendly`. Do not change `SlugService` or add an endpoint. `slugs/new` does not apply `SlugService`'s default-name fallback for a name whose parameterized form fails `Profile::SLUG_PATTERN`. Ordinary names, including the coffee-chat case and reserved words such as Payments, match what create persists.
- A taken slug the user typed by hand. Create still rejects it, the same way a published slug does. This ticket does not auto-suffix a customized slug.
- Managed event types, adhoc, and Callie. Callie slugs are random on the server.

## Code changes

In `app/src/javascripts/features/event_type_editor_crud/`:

1. `tabs/meeting_details/meeting_name_field.js`. `renderUrlSection` returns `LiveUrlPreview` before it looks at `isEditingSlug`, so Edit on a draft does nothing today. Check `isEditingSlug` first, for a draft and for a published event type. Both use the existing `SlugEditField`.

   Keep two pieces of state, and do not put either on the create payload:

   - `slugCustomized`, a form value set only from the slug input's `onChange`. That `onChange` does not call `setNameValue`. `handleSubmit` cannot see component state, which is why this is a form value. Do not add it to `initialValues`. A check result and the Edit seed call `setSlugValue` directly. They do not go through that `onChange`, so they do not set `slugCustomized`.
   - `slugCheck`, a form value: `null`, or `{ name, status }` where `status` is `'ok'` or `'failed'`. `name` is the trimmed meeting name that request was for. It is not `useState`. Meeting details unmounts when another tab opens, and a component-state check would reset and clear the suffixed slug on the way back. The form stays mounted, so this value survives that. It also survives opening Edit, because Edit only toggles `isEditingSlug`. Create does not read it. Do not put `slugCheck` on the create attributes.

   A draft line is confirmed when `slugCustomized` is set, or when `slugCheck.status === 'ok'`, `slugCheck.name` is the current trimmed name, and the form slug is non-empty. The non-empty check matters. The effect below clears the slug when the name leaves a confirmed check. If the user types that same name again before the next blur, `slugCheck` would still say `'ok'` and the line would render as a confirmed empty URL. A confirmed draft uses the slug form value, plus Edit. The `calendly.com/<profile>/` prefix stays in the muted `BodyText`. The slug itself is a sibling in `slugPreviewCss` (`colorTextStandard`), with `data-confirmed="true"`. Do not put that slug inside the muted `BodyText`. The published `CommittedUrlPreview` stays one muted line; `confirmed == null` is that path. Every other draft line is unconfirmed: `deriveSlugFromName(nameValue)` in `slugPlaceholderCss` (`colorTextPlaceholder`), or `your-event` when that derivation is empty, plus the same Edit button, with `data-confirmed="false"`. While the user is still typing, the name does not match `slugCheck`, so the line stays unconfirmed.

   `isEditingSlug` is only ever set to `true`. After Edit opens, `SlugEditField` stays mounted and its value is the form slug. A `useEffect` owns stale-slug cleanup, because the header renames through its own field helper and never enters `handleNameChange`. A clear that lived only in `handleNameChange` would leave `coffee-chat-1` in the form after the header changes the name to "Office Hours", and Edit seeds only when the form slug is empty, so it would open on that old suffix.

   The effect does not clear the form slug while `isEditingSlug` is true. Clearing on a name keystroke blanks the open input, including the input just seeded from an empty slug. Blanking is not the behavior. While Edit is open and `slugCustomized` is not set, and the trimmed name is not a confirmed `slugCheck.name`, set the form slug to `deriveSlugFromName(nameValue)` through `setSlugValue`, not through the input `onChange`. That keeps the open input mirroring the name (`Coffee` becomes `coffee`) instead of sticking on a previous suffix or going empty. Call `setSlugValue` only when that derivation differs from the current form slug, and set `slugCheck` to `null` only when it is not already `null`. An empty name derives to an empty slug. A name that is still the confirmed `slugCheck.name` is left alone, so a slug just returned for this name is not replaced by the local derivation. Once the user types in the slug input, `slugCustomized` is set and this effect stops writing the slug.

   While Edit is closed, the same effect still clears. When the draft is not customized and the current trimmed name is not a confirmed `slugCheck.name`, clear the form slug and set `slugCheck` to `null`. A failed check is not confirmed, so this clear runs once `slugCheck.status` is `'failed'`. Do not clear a slug whose `slugCheck` was just set to `'ok'` for this same name. Skipping the clear whenever Edit is open, and leaving the previous form slug untouched, would keep `coffee-chat-1` in the input after the name changes. The derivation write above is what prevents that.

   Edit opens `SlugEditField`. If the form slug is empty, seed it with `deriveSlugFromName(nameValue)` through `setSlugValue`, not through the input `onChange`. Seeding does not set `slugCustomized`. The effect must not clear that seed on the following render. Opening Edit focuses that input. On a draft, `handleSlugBlur` returns without setting a slug error when `slugCustomized` is not set. Moving from that focused input to the name field blurs an empty derivation, and the required-URL error must not show under the slug the name then fills in. A customized slug still runs the blank, format, and reserved checks on blur.

   `handleNameBlur` still marks the name touched and sets the required error. When `slugCustomized` is set, or the trimmed name is empty, it does not call `slugs/new`. Otherwise it requests the slug. Do not skip that request because Edit is open or because focus is moving into the slug input. `httpRequest` from `@calendly/http-tools` returns `[success, response]`, the same pair `team_form/api.js` uses. Call it with `method: 'GET'`, `url: ${basePath(eventType)}/slugs/new`, and `params: { name }` set to that trimmed meeting name. Do not send `id`. Keep the latest trimmed name and `slugCustomized` in refs the response callback reads. The values captured at the start of the request go stale. Ignore the response when this is no longer the latest request, the ref name is no longer the name that was sent, or the ref says `slugCustomized` is set. A slow response for "Coffee Chat" must not write `coffee-chat-1` after the name is already "Office Hours", and it must not write over a slug the user typed while the request was in flight. On success for the current uncustomized name, in that same handler, `setSlugValue(response.slug)` and set `slugCheck` to `{ name, status: 'ok' }`. The effect has to see those together. If it sees the server slug while `slugCheck` is still unset, it writes the local derivation back over the suffix. That write is intentional while Edit is open. Moving from the name field into the slug input blurs the name, and the response replaces the uncustomized value with the slug `slugs/new` returned, suffix included. Skipping it would leave the open input on the local derivation while publish sends the server slug. On failure for the current name, set `slugCheck` to `{ name, status: 'failed' }`. Do not also clear the form slug in that handler. The effect clears it when Edit is closed, and leaves the derivation in place when Edit is open. `handleNameChange` does not write the name from the slug and does not restore a default name.

2. `draft_slug.js`, next to the CRUD form. `slugErrorMessage` returns the three messages below, or no message. `fetchDraftSlug` is the `GET ${basePath(eventType)}/slugs/new` call. `meeting_name_field.js` and `root_form.js` both import it. Do not copy the check or the request into either file.

3. `form/root_form.js`, the `isCreation` branch of `handleSubmit`, before `createEventType`. Publish stays enabled. `use_editor_primary_actions.js` disables it only while submitting, or when a saved event type is not dirty. A slug error does not disable it, and the draft booking-link schema is empty, so Formik will call `handleSubmit` anyway.

   Use `slugErrorMessage` here. It returns the same three messages `handleSlugBlur` sets, or no message:

   - blank: `publisher.event_type_editor_crud.meeting_details.url_required_error`
   - fails `SLUG_REGEX`: `activerecord.errors.models.event_type_base.attributes.slug.invalid`
   - included in `RESERVED_SLUGS`: `activerecord.errors.models.event_type_base.attributes.slug.exclusion`

   When `slugCustomized` is set and that function returns a message: `form.setFieldTouched('slug', true, false)`, then `form.setFieldError('slug', message)`, then `return Promise.reject()`. The `false` argument matters. `setFieldTouched` validates by default, and this draft schema has no slug, so a validation pass would replace the error with an empty error object. The field renders `slugMeta.error` only when the field is touched. Rejecting matters too. Publish stores `handleCreateSuccess` and runs it when `isSubmitting` falls and the form has no errors (`useHasFormErrors`). `submitForm().catch` drops that callback only when `onSubmit` rejects. A resolved return would publish nothing and still run the success path. Do not toast. Do not call `createEventType`. Do not call `slugs/new`.

   When `slugCustomized` is set and the function returns no message: set `attributes.slug` to the trimmed slug. Do not call `slugs/new`. A customized slug another event type takes between this POST and now still fails on the server. Do not suffix it.

   Otherwise, when the trimmed name is present, call the same GET with the current name and set `attributes.slug` to the returned `slug`. Do not send `values.slug`. It can still be the previous name's slug, or empty because the effect cleared it. A header rename is covered here. When that GET fails, or the name is empty, omit `slug`. The server generates one for a nil slug. That is the one case where a line confirmed by an earlier blur can differ from the saved URL: the blur succeeded, and the publish-time GET then failed. Do not block Publish for it.

   Do not put `slugCustomized` or `slugCheck` on the create attributes.

`getFormatBookingLinkValues` in `booking_link/utils.js` already returns `{}` for a draft (`!id`), and its validation schema returns an empty object in that same case. Leave both, and leave `formatNameValue`, as they are. Adding the slug rule there would also run for the classic editor, whose draft slug is null. The permitted create param is already `:slug` on `event_typeable_params` in `Api::Internal::EventTypeBaseController`. `EventTypes::SlugService.generate` only fills a nil slug, so a slug on this POST is the one that is stored.

Leave `deriveSlugFromName`, the published `handleSlugBlur` messages, `publish_live.js`, and `editor_header_title.js` as they are. Published name blur still does not call `slugs/new`.

## Tests

From `app/`:

```bash
yarn test src/javascripts/features/event_type_editor_crud/tabs/meeting_details/__tests__/meeting_name_field.test.js
```

Extend `meeting_name_field.test.js`. Mock `@calendly/http-tools`. `setupFormField` builds a new `jest.fn()` inside `mockImplementation` and does not return `setValue`. Keep the `setValue` mocks outside that implementation and return them. Otherwise the assertions below cannot see the calls.

Assert:

- A draft with name `Coffee Chat!` shows `coffee-chat` with `data-confirmed="false"` while typing. That slug uses the same class as `your-event`. `setValue` for `name` is not called with a slug or with "New Meeting".
- Blurring that name field calls `GET` `slugs/new` with `params.name` `Coffee Chat!`. The mock resolves to `[true, { slug: 'coffee-chat-1' }]`. `setValue` for `slug` is called with `coffee-chat-1`, the slug text is `data-confirmed="true"`, and its class is not the placeholder class.
- Unmounting the field after that confirmation and mounting it again, with the form values left in place, still shows `coffee-chat-1` with `data-confirmed="true"` and does not call `setValue` for `slug` with `''`. Changing the name after that remount does call `setValue` with `''`, and the line shows the new name's derivation with `data-confirmed="false"`.
- A failed blur (`[false, {}]`) after a confirmed `coffee-chat-1`, with Edit closed, calls `setValue` for `slug` with `''`. The line shows the derivation for the current name with `data-confirmed="false"`. The same failed blur with Edit open does not call `setValue` for `slug` with `''`. If the form slug is still `coffee-chat-1`, that open-Edit failure calls `setValue` with `coffee-chat`.
- A response for "Coffee Chat" that resolves only after the name has changed to "Office Hours" does not call `setValue` for `slug` with `coffee-chat-1`.
- After the slug input is edited, `slugCustomized` is set, and changing the name does not call `setValue` for `slug` again. The line stays on the typed slug.
- Opening Edit on an empty slug seeds the input from the name and does not set `slugCustomized`. With Edit still open, changing the name to `Coffee Chat` calls `setValue` for `slug` with `coffee-chat` and does not call it with `''`. Blurring that slug input before it is customized does not set the required-URL error.
- With Edit open and the slug not customized, blurring the name still calls `setValue` for `slug` with `coffee-chat-1` when the mock resolves to that slug.
- A `slugs/new` response that resolves after the slug input's `onChange` does not call `setValue` with the server slug.
- A published event type (`isCreation: false`) still opens Edit from the button and does not call `slugs/new` on name blur.
- An empty name blur does not call `slugs/new`.

In `form/__tests__/root_form.test.js`, `mockSubmitValues` is what `handleSubmit` receives. The `onSubmit` helper did not pass `setFieldError` before this change. Pass `setFieldError` on that helper. The reserved-slug example below is the assertion that uses it.

- `slugCustomized` unset and `name: 'Coffee Chat'`: the create attributes include the slug from a mocked `[true, { slug: 'coffee-chat-1' }]` and do not include `slugCustomized`.
- `slugCustomized: true` and `slug: 'office-hours'`: the attributes include `office-hours`, and `slugs/new` is not called.
- `slugCustomized: true` and `slug: 'payments'`: `createEventType` is not called, `setFieldTouched` is called as `('slug', true, false)`, and `setFieldError` is called with the reserved-slug message. The submit promise rejects.

## Dev test

Publisher pointed at a backend that serves the existing slug route (`yarn dev:signadot` from `app/`, or a local monolith). `sa_et_crud_single_host` on. Proof the flag is on: the full-page editor with the Meeting details URL under the name field, not the classic side panel.

1. Create a one-on-one. The meeting name input is empty.
2. Type `Coffee Chat`. Before blur, the prefix stays muted and the slug `coffee-chat` is the placeholder color (`colorTextPlaceholder`).
3. Blur the meeting name field. The network panel shows `GET /api/users/<id>/event_types/slugs/new?name=Coffee%20Chat` (or `/api/pages/<id>/...` for a team). The response `slug` is what the slug shows, in the normal text color (`colorTextStandard`). The prefix stays muted. If `coffee-chat` is free, the slug is `coffee-chat`. If it is taken, the slug is `coffee-chat-1`. The meeting name input still reads `Coffee Chat`.
4. On a new draft, click Edit before typing a name, then click the meeting name and type `Coffee Chat`. The slug input shows `coffee-chat` and follows the name. It does not clear, and the required-URL error does not appear under it. Blur the meeting name. The open input changes to the `slugs/new` slug. Type over that slug, change the meeting name, and blur. The typed slug stays after the blur response arrives.
5. Block `slugs/new`. Change the meeting name and blur. The line stays in the placeholder color and shows the new name's derivation. Open Edit. The input is that derivation, not the slug from step 3. With Edit already open, block `slugs/new`, change the meeting name, and blur. The slug input stays on the new name's derivation rather than clearing.
6. Click Edit, change the slug to `office-hours`, then change the meeting name and blur again. The URL stays `office-hours`. Change it to `payments` and click Publish. The create POST is not sent. The reserved-slug error shows on the field. The editor stays on the draft.
7. Set the slug back to `office-hours` and publish. The create POST body includes `slug`. After publish, the event type's booking URL path ends in that slug. Changing the name in the header and publishing without blurring Meeting details still sends the slug `slugs/new` returns for that name.

## Manual check

Flag on (`sa_et_crud_single_host`, or the URL contains `et_editor=new`). The new editor is the proof. A screenshot of the classic editor does not verify this.

- [ ] New one-on-one, flag on. Type a meeting name. Before blur, the URL under the field mirrors that name. The meeting name input is not replaced with the slug or with "New Meeting".
- [ ] Blur the meeting name. With no other event type on that profile using the slug, the URL has no numeric suffix. Create another draft with the same name and blur. The URL ends in the next free suffix from the server (`-1`, then `-2` when `-1` is taken).
- [ ] Edit the URL, then change the meeting name and blur. The edited URL remains. Publish. The live booking link uses that URL.
- [ ] On a draft, Edit the URL to `payments`, then click Publish. The page stays on the draft, the reserved-slug error shows under the URL, and no event type is created. Clear it to empty and click Publish. The required-URL error shows, and no event type is created.
- [ ] Click Edit on an empty draft, then type a meeting name. The slug input mirrors that name and does not clear. Blur the meeting name. The open input shows the `slugs/new` slug. Type a different slug, change the meeting name, and blur. The typed slug stays.
- [ ] With a slug already confirmed, block `slugs/new`, change the meeting name, and blur. The line is the placeholder color and shows the new name's derivation. Edit opens on that derivation, not the previous slug. With Edit already open, the same blocked blur leaves the new derivation in the input.
- [ ] Flag off. A new event type in the classic editor still has no pre-publish slug field. After publish, Edit on the new editor still opens the slug field and still rejects a blank, invalid, or reserved slug.

## Sources

- [SCH-4832](https://calendlyapp.atlassian.net/browse/SCH-4832)
- Acceptance criteria on the ticket: the URL mirrors the name while typing, [SCH-4761](https://calendlyapp.atlassian.net/browse/SCH-4761) so the meeting name is not repopulated, and name blur runs the same uniqueness check publish uses.
- Comment on SCH-4832 (2026-10-21) asking whether a customized URL is an explicit acceptance case. This plan treats the user story ("edit the ET URL" before publish) as that case, and treats mirroring plus the suffix as the behavior when the user has not edited the slug.
- [SCH-4761](https://calendlyapp.atlassian.net/browse/SCH-4761) and `205c5c6338` (`[SCH-4761] fix(event-type-editor): leave meeting name empty for new drafts`).
- Monolith request spec `spec/requests/api/internal/event_types/slug_spec.rb` and `Api::Internal::EventTypes::SlugsController#new`.
