# SCH-4248 — Use radiusMedium on booking-kit time slots

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4248
- Story points: 3
- Repo: `calendly/js-packages`
- Branch: SCH-4248-time-slot-radius
- Pull request: https://github.com/calendly/js-packages/pull/405
- Repo: `calendly/booking`
- Branch: SCH-4248-booking-kit-canary
- Pull request: https://github.com/calendly/booking/pull/518

## Timeline

- Started: 2026-10-14
- Pull request opened: 2026-10-14
- Released: 2026-10-21

## What changed

Selectable time buttons on the one-on-one booking page come from booking-kit, which hardcoded a 4px corner on those spots. The kit now uses the design `radiusMedium` token, and booking was bumped to a published kit version so the booking page actually loads that CSS. Poll flat buttons in booking still use the old 4px styling until a separate follow-up.

## Work

- Replaced hardcoded 4px corners on selectable spot buttons and double-bookable outlines with `radiusMedium` from `@calendly/ui/radius` (`packages/booking-kit/src/spot_list/spot/styles.js`).
- Updated the CSS extractor so `radiusMedium` serializes as the theme variable the booking page already defines, not an invalid fallback name (`scripts/extract-css-from-built.js`).
- Left unavailable slots square, conflict chips at 3px, and the mutual-availability marker circular (`packages/booking-kit/src/spot_list/spot/styles.js`).
- Bumped `@calendly/booking-kit` to 1.0.166 so the booking app picks up the published radius change (`package.json`).
- Removed the yarn patch that pinned styling workarounds for the older booking-kit version (`.yarn/patches/@calendly-booking-kit-npm-1.0.159-85f7792f91.patch`).
