---
name: SCH-4956 Callie meetings booked
overview: Add count_ai_scheduling_assistant_meetings_booked to the Segment identify payload from invitee and event rows. Refresh it on a Callie booking and on a real cancellation, and send 0 only after a host who already had the attribute drops to none.
todos:
  - id: count-query
    content: Add a count of distinct non-canceled event ids for a host, using the Callie invitee signals and the host-email exclusion
    status: pending
  - id: identify-payload
    content: Pass a fresh count as extra traits on Callie book and real cancel, and copy a stored value forward on later identifies
    status: pending
  - id: specs
    content: Cover one meeting with many invitees, cancel to 0, reschedule once, ask_callie_chat excluded, and no trait for a host who never booked via Callie
    status: pending
isProject: false
branch: cursor/sch-4956-callie-meetings-booked
pullRequest: https://github.com/calendly/calendly/pull/39853
started: 2026-10-15
pullRequestOpened: 2026-10-15
released: 2026-10-20
delivery:
  - repo: calendly/calendly
    branch: cursor/sch-4956-callie-meetings-booked
    pullRequest: https://github.com/calendly/calendly/pull/39853
---

# Send the Callie meetings-booked count on identify

Shareable implementation note for [SCH-4956](https://calendlyapp.atlassian.net/browse/SCH-4956) (ET Scheduling · Story · Backlog · P2).

No parent. Related, and out of this change: [DP-7007](https://calendlyapp.atlassian.net/browse/DP-7007) (destinations) and [DP-7147](https://calendlyapp.atlassian.net/browse/DP-7147) (one-time backfill).

Repo: `calendly/calendly`. Local checkout: `/Users/somo.banerjee/Documents/projects/calendly` on `master` at `814a4c1c929`. Default branch is `master`.

```bash
git clone git@github.com:calendly/calendly.git
```

## Goal

Braze and Sprig need an integer user attribute, `count_ai_scheduling_assistant_meetings_booked`, for meetings booked through email Callie. Total Callie invitees is the wrong grain. The count has to come from `invitees` and `events`, not from Segment events.

`Meetings::BookingAnalyticsService#track_booking_success` already enqueues `Analytics::IdentifyUserJob` for `@event.primary_user` with no extra traits. That job calls `AnalyticsService.identify`, which builds the payload in `Analytics::Identify::PayloadService`. A cancellation does not enqueue that job. `Cancellation::Invitee::RunObserversService` and `Cancellation::EventService#run_observers` publish cancel events and do not identify the host.

A reschedule cancels the old event and books a new one. The new booking already identifies the primary host. The old event is `cancelled`, so a count of non-canceled events includes the new meeting once.

## Count

Distinct `events.id` where all of these hold:

- The user has an `event_memberships` row for the event. `User#events` is `through: :event_memberships`, which is the host link the ticket names. This is not `primary_user` alone.
- `events.cancelled` is false.
- At least one invitee on that event has `booking_source = 'ai_scheduling_assistant'` or `utm_source = 'ai_scheduling_assistant'`.
- That invitee's `booking_source` is not `ask_callie_chat`.
- That invitee's email does not match the host's email.

Do not require the invitee to be uncancelled. The ticket excludes canceled meetings, not canceled invitee rows. A group meeting that stays active still counts.

Do not treat thread `utm_content` as attribution. `BookingAnalyticsService#callie_booking?` does, via `OriginExtractorService`, for Invitee Booked. This attribute follows the ticket and the cited `meetings_fct` signals only: `booking_source` and `utm_source`.

`InviteeBase::BOOKING_SOURCE_TYPES` in this checkout does not list `ask_callie_chat`. Keep the exclusion anyway. An invitee with that `booking_source` does not count, even when `utm_source` is `ai_scheduling_assistant`.

Scheduling does not limit `booking_source: ai_scheduling_assistant` to solo event types (`Meetings::SchedulingService#add_booking_attributes`). Round robin and collective meetings count toward every user with an `event_memberships` row on that event. Solo meetings have one such user, the primary host.

## Out of scope

- [DP-7147](https://calendlyapp.atlassian.net/browse/DP-7147). No backfill. A host who has never had this attribute stays without it, including hosts who already have historical Callie meetings.
- [DP-7007](https://calendlyapp.atlassian.net/browse/DP-7007). No Braze, Qualtrics, or Sprig destination work.
- A stored counter column. The read is a count at the moment we identify.
- Caching the count. The acceptance criterion is shortly after the booking or cancellation. `crometrics_data` caches other meeting counts for an hour; do not follow that here.
- Putting the query on every identify. `booking_source` has no index (`db/migrate/20260303232736_add_booking_source_to_booked_invitees.rb` only adds the column). A count on every identify would scan for users who have never used Callie.
- Invitee Booked properties, `callie_booking?`, and `ask_callie_chat` as a new `BOOKING_SOURCE_TYPES` value.
- Identifying co-hosts on a non-Callie booking. The existing primary-host identify on every booking stays as it is.

## Code changes

1. Add a query object under `engines/backend/app/services/analytics/identify/` that returns the integer for one user, using the rules above.

2. In `Analytics::Identify::PayloadService#build_traits`, if `user.identify_data` already has `count_ai_scheduling_assistant_meetings_booked`, copy that integer into the traits. `extra_traits` already merge last, so a fresh count replaces the stored one. A host with no stored value and no extra trait still omits the key. That avoids a Braze data point for everyone else. A later full identify keeps sending the last number instead of dropping the key when `update_user` replaces `identify_data`.

3. In `Meetings::BookingAnalyticsService#track_booking_success`, keep the primary-host identify. When `callie_booking?` is true, pass `count_ai_scheduling_assistant_meetings_booked` for the primary host on that same `perform_async` (the job already takes `extra_traits`). Also enqueue one identify per other `event_memberships` user, with that user's own count. One job per user, not two for the primary host.

4. When a cancellation leaves `event.canceled?` and it is not a reschedule, and any invitee on the event matches the Callie signals above, enqueue `Analytics::IdentifyUserJob` for each `event_memberships` user with that user's recomputed count. Send `0` when the count is `0`. Do this from both cancel observer paths:
   - `Cancellation::Invitee::RunObserversService#run` (`@rescheduling` is already false for a real cancel)
   - `Cancellation::EventService#run_observers`

   An invitee cancel that does not cancel the event does not change this count. Do not identify for this attribute in that case. A reschedule does not identify here; the new booking does.

## Tests

From the calendly repo:

```bash
bundle exec rspec \
  engines/backend/spec/services/analytics/identify/payload_service_spec.rb \
  microservices/meetings/spec/services/meetings/booking_analytics_service_spec.rb
```

Add a spec next to the query object, and extend the cancellation specs that already cover `Cancellation::Invitee::RunObserversService` and `Cancellation::EventService`.

Assert:

- Two Callie invitees on one active event count as 1. A second active event counts as 2.
- A canceled event is excluded. A reschedule (old event canceled, new event active) counts as 1.
- `utm_source = ai_scheduling_assistant` counts. `booking_source = ask_callie_chat` does not, including when `utm_source` is also `ai_scheduling_assistant`.
- An invitee whose email matches the host's email does not qualify that meeting.
- A Callie booking enqueues identify for the primary host with the trait, and for a second membership user with that user's count.
- A non-Callie booking still identifies only the primary host and does not add the trait.
- A real cancel of the host's last Callie meeting enqueues identify with `0` when the trait was already stored. A reschedule cancel does not enqueue an extra identify from the cancel observers.
- `PayloadService` omits the key when it was never stored and no extra trait is passed, and keeps a stored `0`.

## Dev test

Run this after the pull request exists and before requesting review. The RSpec command above is not this check. The booking JSON does not contain the trait. The proof is `users.identify_data`.

1. Put the `staging-overlay-sandbox` label on the pull request. Wait until the Deploy Signadot Sandbox workflow finishes (`.github/workflows/sandbox-deploy.yml`).
2. In the Signadot extension, select that sandbox and open staging until the red sandbox banner is up. Setup is [How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot).
3. In Postman, against that sandbox, `POST /private_api/meetings/invitees` with `Authorization: Bearer <monolith service token>` and `Content-Type: application/json`. Use a staging solo event type's `scheduling_link_uuid` and an open `event.start_time`. Set `invitee.email` to an address that is not the host's. Send `booking_source: "ai_scheduling_assistant"`. Include `book_on_behalf: false` and the required `metadata.remote_ip` and `metadata.url`. The request spec shape is `microservices/meetings/spec/shared_examples/invitee_scheduling.rb`.
4. After the `analytics` Sidekiq queue runs `Analytics::IdentifyUserJob`, read the host:

```ruby
User.find(<host id>).identify_data.dig('traits', 'count_ai_scheduling_assistant_meetings_booked')
```

That value is `1`.

5. Book a second slot the same way. The value is `2`. `PATCH /private_api/meetings/invitees/:uuid/cancel` with `{ "rescheduling": false }` on the second invitee. The value is `1`. Cancel the remaining meeting. The value is `0`.
6. Repeat step 3 with `booking_source` omitted and `tracking.utm_source` omitted, as a host who has never had the key. The key is still absent.
7. One booking with `booking_source: "ask_callie_chat"` and `tracking.utm_source: "ai_scheduling_assistant"` does not increase the count.

## Manual check

No feature flag. Use the email Callie flow, not Ask Callie in the app.

1. Book a meeting through email Callie as a host who has never had the attribute. In the Segment debugger, the host's identify includes `count_ai_scheduling_assistant_meetings_booked` equal to the number of their active Callie meetings, counting that meeting once.
2. Add a second invitee to that same meeting. The number does not go up.
3. Cancel the meeting. The next identify for that host sends `0`.
4. Reschedule a Callie meeting. The host's count stays the same, rather than adding the old and new event.
5. A host with no Callie bookings and no stored attribute does not get the key on an unrelated identify.

## Sources

- [SCH-4956](https://calendlyapp.atlassian.net/browse/SCH-4956)
- [DP-7007](https://calendlyapp.atlassian.net/browse/DP-7007)
- [DP-7147](https://calendlyapp.atlassian.net/browse/DP-7147)
- `meetings_fct` attribution: https://github.com/calendly/calendly_dbt_analytics/blob/e35918d9157ac7a30e0424755c6047e3e14a0bdc/models/calendly_analytics/meetings_fct.sql#L380-L384
- `microservices/meetings/app/services/meetings/booking_analytics_service.rb` (`track_booking_success`, `callie_booking?`)
- `engines/backend/app/services/analytics/identify/payload_service.rb`
- `engines/backend/app/jobs/analytics/identify_user_job.rb`
- `engines/backend/app/services/cancellation/invitee/run_observers_service.rb`
- `engines/backend/app/services/cancellation/event_service.rb`
