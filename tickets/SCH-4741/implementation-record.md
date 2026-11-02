# SCH-4741 — Validate confirmation URLs on blur

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4741
- Type: not recorded
- Story points: not recorded
- Repo: `calendly/mf-publisher`
- Branch: SCH-4741-url-validation-on-blur
- Pull request: https://github.com/calendly/mf-publisher/pull/4124
- Repo: `calendly/calendly`
- Branch: SCH-4741-url-validation-on-blur
- Pull request: https://github.com/calendly/calendly/pull/40304

## Timeline

- Started: 2026-10-25
- Pull request opened: 2026-10-25
- Released: 2026-11-01

## What changed

On the Confirmations section, additional-link and redirect URL fields showed validation errors while the host was still typing or before the field had been touched. Errors now appear after blur or on Done and Publish, while empty URLs still block save.

## Work

- Showed URL validation errors only when the field is touched and not focused, and marked the field touched on blur including empty values (`app/src/javascripts/components/event_type_edit/confirmation_page/url_field.js`).
- Added unit tests for hidden-while-typing, shown-after-blur, and empty required on blur (`app/src/javascripts/components/event_type_edit/confirmation_page/__tests__/url_field.test.js`).
- Added a Storybook story that renders the post-blur invalid URL error without interaction (`app/src/javascripts/components/event_type_edit/confirmation_page/__stories__/url_field.stories.js`).
- Updated confirmation custom-link feature specs to expect the URL required error after blur (`spec/features/event_types/form/et_editor/edit_flow/custom_links_spec.rb`, `spec/features/event_types/form/et_editor/edit_flow/confirmation_page_shared.rb`).
