---
name: SCH-4741 URL validation on blur
overview: Show confirmation URL errors after the field blurs, or on Done and Publish, instead of while the host is typing or before the field has been touched.
todos:
  - id: gate-url-error
    content: Hide UrlField's error until the field has been touched and is no longer focused, and mark it touched on blur including when the value is empty.
    status: pending
  - id: tests
    content: Extend UrlField tests for hidden-while-typing, hidden-before-touch, shown-after-blur, and shown when touched but not focused.
    status: pending
  - id: story
    content: Add a Storybook story that renders the post-blur invalid URL error without a click.
    status: pending
isProject: false
branch: SCH-4741-url-validation-on-blur
pullRequest: https://github.com/calendly/mf-publisher/pull/4124
started: 2026-10-25
pullRequestOpened: 2026-10-25
released: 2026-11-01
delivery:
  - repo: calendly/mf-publisher
    branch: SCH-4741-url-validation-on-blur
    pullRequest: https://github.com/calendly/mf-publisher/pull/4124
  - repo: calendly/calendly
    branch: SCH-4741-url-validation-on-blur
    pullRequest: https://github.com/calendly/calendly/pull/40304
---

# SCH-4741 — Validate confirmation URLs on blur

[SCH-4741](https://calendlyapp.atlassian.net/browse/SCH-4741). Repo: `calendly/mf-publisher`. Checkout: `/Users/somo.banerjee/Documents/projects/SCH-4741/mf-publisher` on `main` (`34f959984f`).

Branch: `SCH-4741-url-validation-on-blur`

Pull request: https://github.com/calendly/mf-publisher/pull/4124

Started: 2026-10-25. Pull request opened: 2026-10-25. Released: 2026-11-01.

This file is the design document and the implementation plan. The new editor is behind Statsig `sa-et-crud-single-host`. The URL field itself is shared with the legacy confirmation panel, so this change is not gated.

## Goal

On the event type Confirmations section, the additional-link URL and the redirect URL show an error while the host is still typing, and they can show one before that field has been touched. The ticket's expected behavior is that the URL is validated on field blur. An empty URL still has to fail on Done and on Publish/Save ([Ryan Guyer's comment](https://calendlyapp.atlassian.net/browse/SCH-4741)).

`useActiveTabValidation` writes schema errors onto the active tab whenever values change. The confirmation schema marks `customLinks[].url` and `redirectConfigurationUrl` required and `url()`-formatted as soon as that section is active. `@calendly/form`'s `TextField` then renders `meta.error` immediately, unless `errorOnInputOnly` is set.

`errorOnInputOnly` is the wrong switch here. `useFormField` sets `touched` on the first keystroke, and `errorOnInputOnly` means `meta.touched && meta.error`. The error would still appear while typing. It would only hide the error before the first input.

`useErrorAfterInitialTouch` clears the first error on a blank, untouched field and then stops. A partial URL is not blank, so the hook does not hide format errors while typing. A later validation pass puts the blank-field error back, which is the "not focused yet" case on the additional-link URL.

## Design

Change the shared `UrlField` so it passes an `error` through to `TextField` only when the field is touched and not focused. On blur, mark it touched even when the value is empty. `useFormField` skips Formik's own blur when the input is empty, so an empty blur would otherwise stay untouched and the required error would stay hidden.

`TextField` applies `fieldProps` after its own `error` prop, and `fieldProps` includes the `error` we pass, so that value is what the input renders. Normalization on blur stays as it is (`normalizeUrl` when there is a value).

This covers the four cases:

- Untouched, including before focus: `touched` is false, so no error is shown even if the schema already failed.
- Focused and typing: `useFormField` has set `touched`, but the field is focused, so the error stays hidden.
- Blur: the field is touched and not focused, so the required or format error shows. A value without a protocol still gains `https://`.
- Done, Publish, or Save: the additional-link Done handler already calls `setFieldTouched` on the URL. Formik's submit, which `@calendly/form` does not replace, marks fields touched when submit validation fails. The field is not focused, so the empty-URL error shows and the save still does not proceed.

Rejected:

- `errorOnInputOnly` on these fields. It still shows the error while typing, because touched is set on change.
- Delaying `useActiveTabValidation` until blur. That hook covers every field on the active tab, and Publish depends on the schema error existing before the host blurs.
- Relaxing the yup schema until blur. Publish could then store an empty redirect URL. The schema stays required.
- Changing `TextField` in `@calendly/form`. That is another repo, and the default would change every text field.

Blast radius: `UrlField` is the confirmation URL input in the new editor (`external_section.js`, `custom_link_card.js`) and the legacy editor (`confirmation_page/external_section.js`, `confirmation_page/custom_link.js`). Both get the same display timing. The link-name field, the meeting slug, and the insert-link popup do not use `UrlField` and stay as they are.

## Out of scope

- [SCH-4742](https://calendlyapp.atlassian.net/browse/SCH-4742), "Change link validation trigger from input to field blur." The title overlaps. This plan does not take that ticket on.
- [SCH-4771](https://calendlyapp.atlassian.net/browse/SCH-4771), URL format validation for Insert Link. That UI is `LinkUrlField`, not `UrlField`.
- The link-name required error, which can still appear as the host types the name.
- The yup rules, the required copy, and `useErrorAfterInitialTouch`. Those stay so the first blank error is still cleared once, and Done/Publish still fail a bad URL.

## Code changes

1. `app/src/javascripts/components/event_type_edit/confirmation_page/url_field.js`. Track focus. Pass `error` only when `meta.touched` is true and the field is not focused. On blur, call `setTouched(true)`, keep the caller's `onBlur` if one is passed, and keep `normalizeUrl` for a non-empty value. Put `error`, `onFocus`, and `onBlur` after the prop spread so a caller's `placeholder` or label still applies and does not drop the gate. No caller passes `onBlur` today.
2. `app/src/javascripts/components/event_type_edit/confirmation_page/__tests__/url_field.test.js`. Cover the cases in Testing. Existing normalization cases stay.
3. `app/src/javascripts/components/event_type_edit/confirmation_page/__stories__/url_field.stories.js`. New story, below.

No procedure page owns this kind of change. Confluence searches for `errorOnInputOnly` plus `validation`, `title ~ "form validation"` in the UX space, `title ~ "Chromatic"` in the Core UX space, `validate on blur` plus `form`, and `inline validation` plus `blur` did not return a form-field error-timing page. The display rule comes from the ticket and from `@calendly/form`'s `TextField` (`packages/form/src/text-field.js` in `calendly/js-packages`): the default renders `meta.error` with no touched check.

Storybook placement follows `docs/STORYBOOK.md`: a component story may sit in `__stories__/`, title `Event Type Edit/Confirmation/UrlField`, and `tags: ['autodocs']`. The story renders the error on its own. It mounts `UrlField` in a `Form` whose schema is `string().url()` with the confirmation message `activerecord.errors.models.custom_link.attributes.url.url`, `initialValues` of an invalid URL, and `initialTouched` for that field. No control has to be clicked. This change is not about dark mode. The repo's Storybook dark-mode toolbar stays the way it is, and this plan does not add a dark story.

`useErrorAfterInitialTouch` stays on the confirmation URL fields. It is also what hides the first blank error on the link name, which this ticket does not change.

## Testing

Local, from `app/`:

```bash
yarn test src/javascripts/components/event_type_edit/confirmation_page/__tests__/url_field.test.js src/javascripts/features/event_type_editor_crud/tabs/confirmation/__tests__/confirmation_tab.test.js
```

`url_field.test.js` asserts:

- An invalid value does not show the URL error before the field is touched.
- Typing an invalid value does not show the error while the input is focused.
- Blurring an invalid value shows the error.
- Blurring an empty required field shows the required error.
- A touched, unfocused field shows the error (the Done and Publish case).
- Blurring `example.com` still sets `https://example.com`.

`confirmation_tab.test.js` already expects the link-name and link-URL required copy after Done on an empty new link. That example keeps passing because Done marks the URL touched and the field is not focused.

Storybook: `yarn storybook` from `app/`, open `Event Type Edit/Confirmation/UrlField`, and confirm the invalid-URL error is already visible. Chromatic on the pull request is the snapshot of that story, not this local open.

Running app, before the diff is committed: clone `calendly/calendly` into this ticket folder if it is not already there, and start it with this `mf-publisher`. In the new editor (`sa-et-crud-single-host` on; the page is the full-page editor and the Confirmations subtitle is "What invitees see after they book."), add an additional link and type in the URL, then set a redirect URL. The error stays hidden while the URL field is focused, appears on blur, and an empty URL still shows its required error on Done or Publish. Repeat the same two fields in the legacy confirmation panel (the flag-off editor). A valid URL still saves.

Shared environment, after the pull request exists: Chromatic snapshots the story when the pull request has the `run-chromatic` label (`.github/workflows/chromatic.yml`). The snapshot is the invalid URL with the error already rendered. Signadot does not apply. Nothing in this change is a request, a response, or Rails behavior, so a staging sandbox of `calendly/calendly` cannot show it. The running editor check stays on the local app.

## Post-production

After the `mf-publisher` production deploy, open Confirmations on a single-host event type. On the flag-on editor, the subtitle "What invitees see after they book." is the proof of which editor is open. Type an invalid additional-link URL and an invalid redirect URL: no error while focused, error after blur. Done on an empty new link, and Publish or Save with an empty redirect URL, still show the required error and do not save. The same URL fields in the legacy panel follow the same timing, because they use `UrlField`.

## Rollback

Revert the `mf-publisher` commit. That restores immediate URL errors. It does not change a redirect or custom link already stored on an event type. Turning off `sa-et-crud-single-host` returns hosts to the legacy editor, and that editor includes this `UrlField` once the bundle is deployed, so the gate is not the undo.

## Sources

- https://calendlyapp.atlassian.net/browse/SCH-4741 — description, Loom https://www.loom.com/share/bb10421c27154f45bbd7405710371129 (no transcript was returned), and Ryan Guyer's comment that an empty value still needs to fail on save.
- Parent [SCH-4200](https://calendlyapp.atlassian.net/browse/SCH-4200) for the new editor path and the `sa-et-crud-single-host` gate.
- `calendly/js-packages` `packages/form/src/text-field.js` and `packages/form/src/use-form-field.js`, read from the default branch, for when `TextField` shows `meta.error` and when touched is set.
- `docs/STORYBOOK.md` and `.github/workflows/chromatic.yml` in this clone. Chromatic is the shared-environment check. Signadot is not, because the change never leaves the client.
- Confluence queries that did not return a form-field error-timing page: `errorOnInputOnly` and `validation`; `title ~ "form validation"` in space UX; `title ~ "Chromatic"` in space FEP; `"validate on blur"` and `form`; `"inline validation"` and `blur`.

## Review

None outside the Scheduling squad. The ticket's squad is Scheduling. The last `CODEOWNERS` match for `app/src/javascripts/components/event_type_edit/` is `@calendly/scheduling-and-availability`, and no later rule matches that path. The confirmation callers live under `app/src/javascripts/features/event_type_editor/` and `app/src/javascripts/features/event_type_editor_crud/`, which have the same owner. No story file is introduced outside that tree.
