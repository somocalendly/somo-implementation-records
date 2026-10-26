# SCH-4956 — Send the Callie meetings-booked count on identify

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4956
- Story points: not recorded
- Repo: `calendly/calendly`
- Branch: cursor/sch-4956-callie-meetings-booked
- Pull request: https://github.com/calendly/calendly/pull/39853

## Timeline

- Started: 2026-10-15
- Pull request opened: 2026-10-15
- Released: 2026-10-20

## What changed

Braze and Sprig needed an integer on Segment identify for how many active meetings a host booked through email Callie. The monolith now computes that count from events and invitees, sends it when Callie books or when a real cancellation zeroes it out, and keeps a stored value on later identifies so the trait does not disappear.

## Work

- Added a service that counts distinct non-canceled events for a host using Callie invitee signals and host-email exclusion (`engines/backend/app/services/analytics/identify/ai_scheduling_assistant_meetings_service.rb`).
- Merged stored and freshly computed `count_ai_scheduling_assistant_meetings_booked` traits in identify payloads (`engines/backend/app/services/analytics/identify/payload_service.rb`).
- Enqueued identify updates with the new count on Callie booking success for the primary host and co-host memberships (`microservices/meetings/app/services/meetings/booking_analytics_service.rb`, `engines/backend/app/jobs/analytics/identify_ai_scheduling_assistant_meetings_job.rb`).
- Re-identified hosts with an updated count after a real cancellation, not after reschedule-only cancels (`engines/backend/app/services/cancellation/invitee/run_observers_service.rb`, `engines/backend/app/services/cancellation/event_service.rb`).
- Added service, job, payload, booking, and cancellation specs for count rules and identify enqueue behavior (`engines/backend/spec/services/analytics/identify/ai_scheduling_assistant_meetings_service_spec.rb`, `engines/backend/spec/jobs/analytics/identify_ai_scheduling_assistant_meetings_job_spec.rb`, `engines/backend/spec/services/analytics/identify/payload_service_spec.rb`, `microservices/meetings/spec/services/meetings/booking_analytics_service_spec.rb`, `engines/backend/spec/services/cancellation/invitee/run_observers_service_spec.rb`, `engines/backend/spec/services/cancellation/event_service_spec.rb`).
