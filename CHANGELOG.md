# Changelog

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
