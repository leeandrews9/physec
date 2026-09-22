# Changelog

## 0.1.3 — 2026-09-22

Applies the first maintainer review of the core enum PR
([ocsf/ocsf-schema#1757](https://github.com/ocsf/ocsf-schema/pull/1757)) to the
two extension enums that carried the same ambiguities. No class shapes or uids
changed.

- **`credential_type_id`: `Card` split into `Proximity Card` (1) and
  `Smart Card` (2).** The old single value hid the distinction that core now
  draws on `factor_type_id`: a proximity card presents a static identifier and
  is not cryptographically authenticated; a smart card is authenticated through
  a cryptographic exchange with the reader. The line is cryptographic
  authentication, not operating frequency or form factor, so a 13.56 MHz card
  read only by its serial number is a proximity card. `Mobile` is renamed
  `Mobile Credential` to match core. **Values 2–7 are renumbered to 3–8**
  (Mobile Credential 3, PIN 4, Biometric 5, License Plate 6, Key 7, Escort 8).
  No known producer populates this attribute yet, so the renumber is taken now
  rather than carrying a coarse value beside two precise ones.
- **`dispatch_outcome_id`: `Responded Unfounded` (5) narrowed; `Responded
  Unsubstantiated` (7) added.** Value 5 now means a resource arrived and
  established that the condition was not genuine. A resource that arrived and
  could neither confirm nor rule out the condition is the new value 7, so
  consumers do not read an inconclusive visit as a confirmed false alarm. In a
  production corpus from a US guarding and monitoring operation, all 66
  "no event" dispatch outcomes carry no identified false-alarm cause; under
  the old definition every one of them was emitted as unfounded. Values 1–6
  are unchanged.

## 0.1.2 — 2026-09-09

Rename `security_zone` → `site_zone`; tier crosswalk guidance; no uid or enum
changes.

- **`security_zone` → `site_zone`** (object + attribute). "Security zone" already
  means a network zone in firewalls and DNS appliances (raised on the OCSF
  community call of 2026-09-08); `site_zone` sits beside `site_uid` and
  `site_name`. References updated on `alarm_point`, `checkpoint`, `portal`,
  `post`, the base class, the `physical_site` profile, and the examples.
  Observable type id 42003 is unchanged.
- `zone_type_id` description now states that the six values are generic tiers
  by security posture and that regime-specific tiers (DoD, DOE, NRC areas) map
  onto them rather than extending the list, with the native tier name carried
  in `zone_type`. Enum values unchanged.

## 0.1.1 — 2026-08-31

Applies the guidance from the OCSF mappings call of 2026-08-20 (Zach, Hunter):
ride core classes with a profile where possible, and never shadow a core
dictionary name. No class shapes, uids, or activity enums changed.

### Renamed (core dictionary collisions removed)

- **`zone` → `security_zone`** (object + attribute). Core's `zone` dictionary
  attribute means a network zone or LAN segment (on `device`); this extension
  was silently retyping it. References updated on `alarm_point`, `checkpoint`,
  `portal`, `post`, the base class, and the `physical_site` profile.
- **`disposition` → `disposition_details`** (object + attribute). Core's
  `disposition`/`disposition_id` are the findings verdict pair. Renamed, an
  event can carry the core pair and this richer object side by side.
  References updated on Alarm Disposition, Incident Report, and Response
  Activity.
- **`credential` → `physical_credential`** (object + attribute), with its
  intended shrinkage documented in the object description: access grant/deny
  decisions should ride core Authentication + the `physical_site` profile
  (badge/prox/smart-card values to be proposed on the core `auth_factor`
  enum); the object remains for door-state events that are not
  authentication attempts.

### Changed

- The site/building/floor additions to core `location` are now gated behind
  the `physical_site` profile instead of patching the core object
  unconditionally. Every core `device` inherits physical site context via
  `device.location` when the profile is applied.

### Added

- `incident_type_common` vocabulary value **`unclassified`** (24 → 25
  values). Production measurement showed 15% of reports arrive with no
  classifying flag; without this value they were silently dropped from the
  event stream.

## 0.1.0 — 2026-08-15

Initial draft release: one category (`physical_security`, 4230), seven event
classes, ten objects, `physical_site` profile, `incident_type_common`
vocabulary, synthetic example chains, CI validation against upstream OCSF.
