---
name: CRM-5970 invitee question uuids
overview: Give every invitee-form question a durable uuid, return it from the internal and meetings private APIs, then expose the same value on public event type custom_questions. The editor keeps saving by the integer id.
todos:
  - id: column
    content: Add a nullable unique uuid column on custom_fields, include UuidConcern on CustomField, and backfill existing rows
    status: pending
  - id: clone
    content: Exclude uuid from EventTypeable::Clone::AttributesService so a duplicated event type gets new question uuids
    status: pending
  - id: private-internal
    content: Return uuid from EventTypes::CustomFieldPresenter and Meetings::EventTypeAssociation::CustomFieldBlueprint, keeping the integer id
    status: pending
  - id: public
    content: Add uuid to ApiV2::EventTypeCustomFieldBlueprint and to EventType.custom_questions in the bundled OpenAPI schema
    status: pending
  - id: not-null
    content: SCH-5088, after the SCH-5087 backfill has run in production, makes custom_fields.uuid NOT NULL and returns uuid from the APIs
    status: pending
  - id: specs
    content: Cover backfill, model generation, clone uniqueness, and stable uuid across an internal rename or reorder
    status: pending
isProject: false
---

# Invitee question uuids for the public event type API

Two Scheduling stories. [SCH-5087](https://calendlyapp.atlassian.net/browse/SCH-5087) stores the uuid and runs the backfill. [SCH-5088](https://calendlyapp.atlassian.net/browse/SCH-5088) makes the column `NOT NULL` and returns the uuid from the APIs. SCH-5087 blocks SCH-5088. SCH-5088 starts after the backfill has finished in production.

Contacts placeholder: [CRM-5970](https://calendlyapp.atlassian.net/browse/CRM-5970), linked to SCH-5087. Contract: [ECO-2936](https://calendlyapp.atlassian.net/browse/ECO-2936).

Repo: `calendly/calendly`. Default branch is `master`. The event type editor that saves questions lives in `calendly/mf-publisher` and does not change.

```bash
git clone git@github.com:calendly/calendly.git
```

## Goal

A booking question that already exists gets a `uuid` that stays the same when a host edits the text, type, choices, or order. A new question gets one on create. A duplicated event type gets new question uuids.

`GET /api/v2/event_types` and `GET /api/v2/event_types/{uuid}` include `custom_questions[].uuid`. Contacts can later pass that value as `question_uuid` on `POST /contacts/custom_field_definitions`. This ticket does not build that POST.

## What “done” means

- Every `custom_fields` row has a unique `uuid` after [SCH-5087](https://calendlyapp.atlassian.net/browse/SCH-5087). [SCH-5088](https://calendlyapp.atlassian.net/browse/SCH-5088) makes the column `NOT NULL` and returns `uuid` from the APIs, after that backfill has finished in production.
- The internal app API and the meetings private API return that `uuid` next to the existing integer `id`.
- The public event type question object includes `uuid`.
- Renaming or reordering a question through the editor returns the same public `uuid`.
- Cloning an event type does not reuse the source question uuid.

## 1. Store the uuid

`custom_fields` has no uuid column. Follow the `workflow_rules` pattern.

1. Add a nullable `uuid` column and a concurrent unique index where `uuid IS NOT NULL`.
2. Include `UuidConcern` on `CustomField`. New rows get `SecureRandom.uuid` in `after_initialize`. That is the same generator event types use, so the public value is a normal hyphenated UUID.
3. Backfill existing rows with `db/backfill/custom_fields_uuid.rb` (`SELECT id FROM custom_fields WHERE uuid IS NULL`, then set `SecureRandom.uuid` per row) and a `Backfiller.run` migration. `custom_fields` is large, so this stays out of a transaction, same as `Backfill::WorkflowRulesUuid`.
4. [SCH-5088](https://calendlyapp.atlassian.net/browse/SCH-5088) makes the column `NOT NULL` after this backfill has finished in production. That migration stays out of SCH-5087. Override `require_uuid?` to `new_record? || uuid_changed?` on this ticket, the same override `AvailabilityRule` and `EventTypeMembership` use, so an edit of a question the backfill has not reached yet still saves.

Do this before any API returns the field. A response that sometimes omits `uuid` is worse than waiting one deploy.

## 2. Keep copies from sharing a uuid

`EventTypeable::Clone::AttributesService` copies every column except `id`, `event_type_id`, `created_at`, and `updated_at`. Add `uuid` to that except list. The new row then gets its own uuid from `UuidConcern`. Without that, duplicating an event type hits the unique index, or two event types advertise the same question id.

The editor does not need a change. In `mf-publisher`, `formatQuestionId` (`app/src/javascripts/components/event_type_edit/invitee_questions/format_values_to_server_data.js`) sends the integer `id` for a saved question and omits it only for a question whose id is null or starts with `new`. Edit and reorder update the same row. Delete sends that `id` with `_destroy: true`. Save is `PATCH /api/users/{user_id}/event_types/{id}` (`memberPath` in `event_type_path_utils.js`). That route is `scope module: :internal` in `config/routes.rb`, so the path has no `/internal` segment.

## 3. Return it from the private and internal APIs

The ticket asks for the private API first, because that is the in-product representation of the question. Two responses already serialize it:

- Internal app API, which the editor reads: add `uuid` to `EventTypes::CustomFieldPresenter` (`engines/backend/app/presenters/event_types/custom_field_presenter.rb`). It already returns the integer `id`. Keep `id`. The editor matches questions by that integer. The same presenter is the question list on the event type item, the event type list, managed event types, and the dashboard event type payload. Those responses gain `uuid` too.
- Meetings private API: add `field :uuid` to `Meetings::EventTypeAssociation::CustomFieldBlueprint`. It also keeps `identifier :id`. The meetings event type blueprint uses this blueprint on the default view and on the `marketplace_apps` view (`microservices/meetings/app/blueprints/meetings/event_type_blueprint.rb`). Both gain `uuid`. The JSON key stays `custom_fields`.

Neither request body changes. Clients that ignore unknown keys are unaffected.

Leave these question payloads unchanged. They are the same row, and this ticket does not publish `uuid` on them:

- `LegacyApiV2::EventTypeBlueprint` renders `custom_fields` through `LegacyApiV2::CustomFieldBlueprint`. That is the legacy event type resource, not `custom_questions`.
- `PrivateApi::EventTypeCustomFieldBlueprint` is the poll question list on `/private_api/facade`. It already returns `custom_questions` plus the integer `id`. Public polls use `ApiV2::EventTypeCustomFieldBlueprint` and do gain `uuid`.
- `Api::Internal::CustomFieldBlueprint` has no caller.
- `Meetings::PubSub::EventBlueprint` sends `custom_fields` as `{ id, position }` only.

## 4. Expose it on the public API

Add `field :uuid` to `ApiV2::EventTypeCustomFieldBlueprint`. Both `GET /event_types` and `GET /event_types/{uuid}` use that blueprint, so both gain `custom_questions[].uuid`. Polls use it too, so a poll question gains the same field. That is the same row and the same id.

`apps/api_v2/api_docs/openapi-bundled.yaml` describes `EventType.custom_questions[]` with `name`, `type`, `position`, `enabled`, `required`, `answer_choices`, and `include_other`. That object does not set `additionalProperties: false`. Committee request specs still load this file (`apps/api_v2/spec/spec_helper.rb`, `strict: true`). Add `uuid` (string) to that question schema in this change so the bundled contract matches the response. The published developer reference in the separate `api-docs` repo gets the same additive field. The contract’s published shape is that reference.

## 5. Tests

`spec/backfill/workflow_rules_uuid_spec.rb` is the pattern, and both of its examples are `xit`. The new spec has to actually run.

```bash
bundle exec rspec \
  spec/backfill/custom_fields_uuid_spec.rb \
  engines/backend/spec/models/custom_field_spec.rb \
  engines/backend/spec/services/event_typeable/clone/attributes_service_spec.rb \
  spec/requests/api/internal/event_types_spec.rb \
  microservices/meetings/spec/blueprints/meetings/event_type_blueprint_spec.rb \
  apps/api_v2/spec/blueprints/event_type_blueprint_spec.rb \
  apps/api_v2/spec/requests/event_types/event_types_show_spec.rb \
  apps/api_v2/spec/requests/event_types/event_types_index_spec.rb
```

- Backfill spec: a row with a null uuid is filled, a row that already has one is left alone.
- Model: a new `CustomField` has a uuid before save, and a reload keeps it.
- Clone spec: the copy’s question uuid differs from the source.
- Internal event type show, meetings event type blueprint, and public event type show/index/blueprint specs: the question object includes `uuid`, and an update that sends the existing integer `id` returns the same `uuid`.
- Public request spec: reordering or renaming through `PATCH /api/users/{user_id}/event_types/{id}` does not change `uuid`.

## Out of scope

- `POST /contacts/custom_field_definitions` and the MCP tool. Contacts consumes the uuid this ticket publishes.
- The editor payload in `mf-publisher`.
- The `NOT NULL` migration and every API that returns `uuid`. Those are [SCH-5088](https://calendlyapp.atlassian.net/browse/SCH-5088), blocked by SCH-5087.
- Outbound webhook and extension-API copies of the question object. They are separate customer surfaces. Adding `uuid` there changes those contracts and is not required for CRM-5970.
- Legacy API v2 `custom_fields`, private API facade poll `custom_questions`, the unused `Api::Internal::CustomFieldBlueprint`, and the meetings pub/sub `{ id, position }` payload. Listed under section 3.
- Replacing the integer `id` anywhere. The editor, the meetings private API, and oap-api’s `questionId` (`sch_custom_field_id`) keep using it. The public uuid is a second identifier on the same row.

## Dev test

The engineer runs this after the pull request exists and before requesting review. RSpec passing is not this check.

1. Add the `staging-overlay-sandbox` label so `.github/workflows/sandbox-deploy.yml` deploys a Signadot sandbox. Route staging with the Signadot extension ([How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot)).
2. Use an event type that already has a booking question. With the editor session, `GET /api/users/{user_id}/event_types/{id}`. Record `custom_fields[].id` and `custom_fields[].uuid` for that question.
3. With a token that can read event types, `GET /api/v2/event_types/{event_type_uuid}`. Proof is `custom_questions[].uuid` equal to the hyphenated UUID recorded in step 2.
4. `PATCH /api/users/{user_id}/event_types/{id}` with `custom_fields_attributes: [{ id: <integer from step 2>, name: "<new text>", position: <new position> }]`. Repeat the public GET. Proof: that question’s `uuid` is unchanged, and `name` and `position` are the new values.
5. PATCH again with one more question that omits `id`. Public GET. Proof: the new question has a different `uuid`, and the original `uuid` is unchanged.
6. `POST /api/users/{user_id}/event_types/{id}/clone`, then `GET /api/v2/event_types/{copy_uuid}`. Proof: the copy’s question uuids differ from the source.
7. `GET /private_api/meetings/event_types/{uuid}` with the meetings service token. Proof: `custom_fields[].uuid` matches the public uuid, and `id` is still the integer.

## Manual check

QA, after handoff:

- On an event type that already has a booking question, `GET /api/v2/event_types/{uuid}` shows `custom_questions[].uuid`.
- In the event type editor, rename that question and move it. Save. The public `uuid` is unchanged.
- Add a question in the editor. The new question has a different public `uuid`.
- Delete a question in the editor. That `uuid` is no longer on the public event type.
- Duplicate the event type. The copy’s question uuids differ from the source.
- An outbound webhook event type payload and an extension API event type still omit `uuid`.

## Sources

- [CRM-5970](https://calendlyapp.atlassian.net/browse/CRM-5970). No comments. Description asks for a uuid on invitee-form questions, on the private API first, then the public API. Parent [CRM-5830](https://calendlyapp.atlassian.net/browse/CRM-5830).
- Contract [ECO-2936](https://calendlyapp.atlassian.net/browse/ECO-2936) and [Mike Kimelblat’s draft](https://docs.google.com/document/d/1YhdLPvzk9MdfJOCll5KpFgrIa4yRc65tS2H7fABUavE/edit?tab=t.0). Garima’s question on that doc is the stable question id: `custom_questions[].uuid` on `GET /event_types` and `GET /event_types/{uuid}`, unique, and unchanged when the question text or order changes. `POST /contacts/custom_field_definitions` with `question_uuid` is the consumer, and it is not this ticket.
- `mf-publisher` `format_values_to_server_data.js`, re-read for this check. A saved question always sends its integer `id`.
