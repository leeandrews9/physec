# Changelog

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
