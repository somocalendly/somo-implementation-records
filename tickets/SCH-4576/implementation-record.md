# SCH-4576 — Return 4XX when `event_fields` is an array of strings

## Metadata

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4576
- Repo: not named. The change is `app/controllers/booking/api/invitees_controller.rb` on `master`.
- Branch: not recorded
- Pull request: not recorded
- Parent: [SCH-3652](https://calendlyapp.atlassian.net/browse/SCH-3652)
- Squad / priority / points: Scheduling · P2 · 3 SP · Low impact
- Status: Released

## Timeline

- Started: not recorded
- Pull request opened: not recorded
- Released: 2026-10-08

## Story

A booking `POST` that sends `event_fields` as an array of UUID strings returns 500. `prepare_params` treats each entry as a Hash and calls `field.delete(:include_other)`. A string does not accept a symbol, so that call raises `TypeError`. The fix checks that each entry is a Hash before that call and returns the controller’s existing 4XX validation response. No booking is created. A real answer object with `id`, `value`, and `include_other` still books.

This is the same production error as [SCH-3978](https://calendlyapp.atlassian.net/browse/SCH-3978) (Won't Do). Datadog issue `13ac0374-de11-11f0-92fb-da7ad0900000`.

Left alone: the client that sends the strings, Book on Behalf key allowlisting ([SCH-4730](https://calendlyapp.atlassian.net/browse/SCH-4730)), and client-injected field persistence ([SCH-4108](https://calendlyapp.atlassian.net/browse/SCH-4108), [SCH-4109](https://calendlyapp.atlassian.net/browse/SCH-4109)). Do not reject unknown keys and do not strip `id`.

## Code

- `app/controllers/booking/api/invitees_controller.rb`, inside `prepare_params`, before `field.delete(:include_other)`: if an entry is not a Hash, return the controller’s existing invalid-params response. Keep stripping `include_other` for real hashes.
- Do not add an allowlist of `name` and `value`. Do not add a new validator class unless `delete(:include_other)` on `event_fields` shows up at a second call site.
- A request spec covers an array of UUID strings, nil, a bare string, and a mix of string and hash. All of those are 4XX with no invitee saved. A valid `id` / `value` / `include_other` object still books.

## Proof and rollback

Replay the array-of-UUID-strings payload against the booking invitees endpoint and get 4XX, not 500. Book with a real custom-question object and succeed. The Datadog issue stops reporting this exception.

The plan does not record that this check was run. It does not name a rollback.

## People

The plan does not name an outside reviewer. SCH-4576’s description never added the route. The file, line, and payload come from SCH-3978.

## Still open

Branch and pull request are not in the plan. Confirming the line number on current `master` is called out in the plan and is not recorded as done.

Plan: tickets/SCH-4576/sch_4576_implementation_plan.plan.md
