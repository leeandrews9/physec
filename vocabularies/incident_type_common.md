# incident_type_common — controlled vocabulary

**Version 0.1.0 · Published** · the vocabulary behind the required
`incident_type_common` attribute of Incident Report Activity (4230007).
Machine-readable form: [`incident_type_common.json`](incident_type_common.json).

**Governance:** values are append-only. Renaming or removing a value is a
breaking change and requires a major version. New values may be added in
minor versions. Emitters MUST use these tokens verbatim; locally defined
types belong in `incident_type_internal` alongside, never in place of, a
common value.

**Crosswalk status:** the APCO, NIBRS, and NFIRS columns are structurally
present but **unverified** in v0.1 — they will be populated from licensed
copies of APCO/NENA ANS 2.103.2, NIBRS, and the NFIRS 700-series and marked
verified per row in a minor release. Do not rely on them yet; publishing an
unverified crosswalk as authoritative is exactly what this schema refuses
to do.

| Token | Caption | Description | APCO 2.103.2 | NIBRS | NFIRS-700 |
|---|---|---|---|---|---|
| `suspicious_activity` | Suspicious Activity | Person, vehicle, or circumstance warranting attention; no confirmed offense, no confirmed unauthorized presence. Confirmed presence → `trespass`. | unverified | unverified | unverified |
| `trespass` | Trespass | Confirmed unauthorized presence, incl. refusal to leave and encampment. Control defeated → `access_violation`. | unverified | unverified | unverified |
| `access_violation` | Access Violation | An access control circumvented or misused (tailgating, credential misuse, unauthorized portal entry). The control's defeat is the defining feature. | unverified | unverified | unverified |
| `insecure_opening` | Insecure Opening | Door/gate/window found unsecured, no evidence of force. Force evident → `burglary_attempt`. | unverified | unverified | unverified |
| `theft` | Theft | Confirmed taking of property (cargo, materials, equipment, fuel, wire, from vehicles). Unsuccessful entry to steal → `burglary_attempt`. | unverified | unverified | unverified |
| `burglary_attempt` | Burglary / Forced Entry Attempt | Attempted or completed forced entry, property taken or not. Force distinguishes it from `insecure_opening`. | unverified | unverified | unverified |
| `vandalism` | Vandalism | Deliberate damage or defacement where taking property was not the object. | unverified | unverified | unverified |
| `disturbance` | Disturbance | Verbal altercation, threats, nuisance or disorderly conduct without physical violence. Contact → `assault`; weapon present → `weapon`. | unverified | unverified | unverified |
| `weapon` | Weapon Observed | A weapon observed, mentioned, or brandished; takes precedence over `disturbance`/`trespass` when present. | unverified | unverified | unverified |
| `assault` | Assault | Physical violence against a person; supersedes `disturbance` once contact occurs. | unverified | unverified | unverified |
| `fire` | Fire / Smoke | Fire, smoke, or combustion event of any size, confirmed or reasonably suspected. | unverified | unverified | unverified |
| `evacuation` | Evacuation | An evacuation or shelter-in-place of any scope, whatever the trigger. | unverified | unverified | unverified |
| `medical` | Medical Emergency | Medical emergency or injury requiring aid, from on-site first aid to EMS transport. | unverified | unverified | unverified |
| `safety_hazard` | Safety Hazard | Immediate hazard to persons requiring prompt mitigation. Non-immediate deficiencies → `property_maintenance`. | unverified | unverified | unverified |
| `utility_failure` | Utility Failure | Utility loss or failure creating an unsafe or materially degraded condition. | unverified | unverified | unverified |
| `property_maintenance` | Property / Maintenance Issue | Non-immediate facility deficiency (lighting, leaks, broken fixtures, sanitation, elevators). Immediate danger → `safety_hazard`. | unverified | unverified | unverified |
| `security_device_fault` | Security Device Fault | Camera, intercom, access controller, sensor, or barrier found faulty, offline, or damaged. | unverified | unverified | unverified |
| `parking_violation` | Parking Violation | Vehicle parked or stored against site rules, incl. abandoned/tow-eligible; no damage or injury. | unverified | unverified | unverified |
| `vehicle_incident` | Vehicle Incident | Vehicle event with damage, injury risk, or traffic hazard (collision, struck fixture, dangerous operation). | unverified | unverified | unverified |
| `service_assist` | Service / Assist | Routine assistance to a customer, tenant, or visitor; no adverse security condition. | unverified | unverified | unverified |
| `welfare_check` | Welfare Check | A check on a person's wellbeing, requested or self-initiated. | unverified | unverified | unverified |
| `false_alarm` | False Alarm / Unfounded | Investigated and determined to reflect no actual condition, where no other category applies. Paperwork artifacts → `administrative`. | unverified | unverified | unverified |
| `administrative` | Administrative | Documentation-only records: duplicates, tests, training, information-only. | unverified | unverified | unverified |
| `unexpected_occupancy` | Unexpected Occupancy | Person or activity present outside schedule or authorization without confirmed trespass intent. Confirmed unauthorized → `trespass`. | unverified | unverified | unverified |

## Provenance

This vocabulary was not designed at a whiteboard. It is the distillation of a
live 44-flag operational taxonomy running in production at a US physical
security firm, where every flag has been crosswalked to one of these 24
values and exercised by real incident and dispatch reports since 2026-08 —
first through an interim disposition crosswalk, then through a structured
verification labeling layer feeding a live physec emitter. Flags too
granular for cross-organization comparability (catalytic converter theft,
copper theft, gate tailgating) map into these values and carry their detail
in `incident_type_internal` and the method-of-entry tagging instead.
