# CRM-5970 — Invitee question uuids for the public event type API

## Metadata

- Ticket: https://calendlyapp.atlassian.net/browse/CRM-5970
- Repo: `calendly/calendly`
- Branch: not recorded
- Pull request: not recorded
- Parent: [CRM-5830](https://calendlyapp.atlassian.net/browse/CRM-5830)
- Related: [SCH-5087](https://calendlyapp.atlassian.net/browse/SCH-5087) stores the uuid and runs the backfill. [SCH-5088](https://calendlyapp.atlassian.net/browse/SCH-5088) makes the column `NOT NULL` and returns the uuid from the APIs after that backfill. Contract: [ECO-2936](https://calendlyapp.atlassian.net/browse/ECO-2936).
- Squad / priority / points: not recorded
- Status: Released

## Timeline

- Started: not recorded
- Pull request opened: not recorded
- Released: 2026-10-16

## Story

Contacts needs a stable id for a booking question. CRM-5970 is the contacts placeholder. A question that already exists gets a `uuid` that stays the same when a host edits the text, type, choices, or order. A new question gets one on create. A duplicated event type gets new question uuids. SCH-5087 is the column, the model, and the backfill. SCH-5088 is the `NOT NULL` constraint and returning `uuid` from the APIs, and it starts after the backfill has finished in production. The editor keeps saving by the integer `id`. This does not build `POST /contacts/custom_field_definitions`.

Left alone: the event type editor in `mf-publisher`, outbound webhooks, the extension API, legacy API v2 `custom_fields`, the private API facade poll questions, the meetings pub/sub `{ id, position }` payload, and replacing the integer `id` anywhere.

## Code

- Add a nullable unique `uuid` on `custom_fields`, include `UuidConcern` on `CustomField`, and backfill existing rows outside a transaction. New rows get `SecureRandom.uuid`.
- Exclude `uuid` from `EventTypeable::Clone::AttributesService` so a duplicated event type does not reuse the source question uuid.
- Return `uuid` beside the integer `id` from `EventTypes::CustomFieldPresenter` and `Meetings::EventTypeAssociation::CustomFieldBlueprint`.
- Add `uuid` to `ApiV2::EventTypeCustomFieldBlueprint` and to `EventType.custom_questions` in `apps/api_v2/api_docs/openapi-bundled.yaml`.
- The `NOT NULL` migration stays on SCH-5088. The plan says a response that sometimes omits `uuid` is worse than waiting for that deploy.

## Proof and rollback

On a Signadot sandbox, an existing question’s public `custom_questions[].uuid` matches the internal `custom_fields[].uuid`. Renaming or reordering through the editor keeps that uuid. A new question gets a different one. A cloned event type gets different question uuids. The meetings private API returns the same uuid and still returns the integer `id`.

The plan does not record that this check was run. It does not name a rollback.

## People

The plan does not name an outside GitHub team. The contract context is ECO-2936. Garima’s question on the contract draft is the stable public id.

## Still open

Branch, pull request, and whether the production backfill has finished are not in the plan. SCH-5088 is the follow-on.

Plan: tickets/CRM-5970/crm_5970_implementation_plan.plan.md
