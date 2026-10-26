# SCH-4576 — Return 4XX when `event_fields` is an array of strings

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4576
- Story points: 3
- Repo: `calendly/booking`
- Branch: SCH-4576-reject-invalid-event-fields
- Pull request: https://github.com/calendly/booking/pull/514

## Timeline

- Started: 2026-10-09
- Pull request opened: 2026-10-09
- Released: 2026-10-12

## What changed

A booking request that sent custom question answers as a bare list of UUID strings hit `String#delete` instead of hash logic and returned 500. Invalid entries now fail validation with the same 4XX pattern as other bad invitee params, with no booking created. Legitimate objects with `id`, `value`, and `include_other` still work.

## Work

- Rejected non-hash entries in `event_fields` before stripping `include_other`, using the controller’s existing invalid-params response (`app/controllers/booking/api/invitees_controller.rb`).
- Introduced a dedicated validation error type for malformed `event_fields` payloads (`app/errors/booking/api/invalid_event_fields_error.rb`).
- Added request specs for string-only arrays, mixed invalid entries, and valid hash objects that still book (`spec/requests/api/invitee_create_spec.rb`, `spec/requests/api/invitee_async_create_spec.rb`, `spec/requests/api/invitee_update_spec.rb`).
