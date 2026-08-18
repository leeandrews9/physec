# physec — an OCSF extension for physical security operations

**v0.1.0 · draft · not yet submitted to OCSF**

`physec` is an open event schema for physical security operations: access control decisions, alarm signals and their disposition, guard patrols, post staffing, response dispatch, and incident reports. As of v0.1 the required `incident_type_common` attribute is backed by a [published 24-value controlled vocabulary](vocabularies/incident_type_common.md) derived from a live production taxonomy.

**physec is emitted in production.** An operational security firm (AGS Pro, Inc.) runs five live emitter views over its incident ontology — patrol, post staffing, alarm disposition, response, and incident report activity — producing events that validate against this schema; an instance-validation suite checks every emitted event's required attributes, enum ids, and type_uid arithmetic against these definitions.

It is built as an extension to the [Open Cybersecurity Schema Framework](https://ocsf.io/) rather than as a new standard, so it inherits OCSF's governance, tooling, and validation — and so that physical events land in the same normalized event stream as logical ones.

## Why this exists

Physical security has good standards at the device layer and none at the operational layer.

- **ONVIF** standardizes device metadata and door/access/credential topics. It stops at the device boundary.
- **OSDP** (IEC 60839-11-5) standardizes reader-to-controller messaging. It carries no access decision at all — that lives in the controller.
- **SIA DC-03** and **Ademco Contact ID** standardize alarm signalling codes. They describe signals, not what the signals turned out to be.
- **ANSI/TMA AVS-01** standardizes the alarm validation score. It is not represented in any machine-readable event schema.
- **Guard patrols, post staffing, and daily activity reporting have no standard whatsoever.** Every vendor invented its own shape.

And OCSF itself — the live, well-governed open schema for security telemetry — currently has **no physical security category**. Its eight categories are System, Findings, IAM, Network, Discovery, Application, Remediation, and Unmanned Systems.

So the layer where a security operation actually runs — *what was reported, what it turned out to be, who responded, how long it took, and what it cost* — is unmodelled. That is the gap this fills.

## Design principles

**1. Don't invent what already exists.** Every event class carries a `signal_code` object crosswalking to SIA DC-03, Contact ID, ONVIF topics, and OSDP replies, plus the originating system's verbatim string in `native_type`. Where a vocabulary exists, this schema adopts it:

| Concept | Adopted from |
|---|---|
| Alarm validation score (Level 0–4) | ANSI/TMA AVS-01-2024 |
| Verification method | ANSI/TMA CS-V-01-2022 |
| Urgency / certainty / response type | OASIS CAP 1.2 (ITU-T X.1303) |
| Door, access, and credential semantics | ONVIF Access Control / Door Control / Credential service specs |
| Signal codes | SIA DC-03, Ademco Contact ID (DC-05) |
| Dual incident coding | APCO/NENA EIDD (ANS 2.105.1) |
| Offense codes | NIBRS (optional crosswalk) |

**2. Separate the event from its state transition.** SIA encodes this in the second letter of its two-character codes; Contact ID encodes it in the qualifier field. This schema follows both: `lifecycle_id` carries new / restore / still-present / cancel / trouble / bypass / test, so `Portal Forced` and `Portal Forced Restored` are one activity in two lifecycle states, not two activities.

**3. Separate the decision from the physical outcome.** ONVIF distinguishes `AccessGranted` (authorized), `AccessTaken` (portal used), and `AccessNotTaken` (authorized but unused). Preserving that distinction is what makes tailgating and no-entry-made conditions expressible at all.

**4. Inferred conditions must declare their derivation.** No access control system emits a native tailgate event — it is always inferred, from sensor fusion, video analytics, or grant-count reconciliation. Any such event requires `derivation_id` and should carry `confidence_pct`. This is deliberate: an inferred metric without a stated derivation is a marketing claim, not data.

**5. Disposition is a first-class event, not a status field.** `Alarm Disposition Activity` (class 4230003) is the point of the whole schema. A signal without a disposition is unlabelled data. A signal *with* one is a training example, an audit record, and a benchmark row.

**6. Represent human, remote, and robotic responders uniformly.** `response_resource` covers on-site officers, mobile patrols, remote operators, automated talkdown, drones, ground robots, and public agencies with the same four timestamps. That is what makes response paths comparable for the same class of event.

## Contents

**One category** — `physical_security`, effective `category_uid` **4230**.

**Seven event classes:**

| `class_uid` | Class | Status of the ground it covers |
|---|---|---|
| 4230001 | Physical Access Activity | Well covered by ONVIF; here for interop |
| 4230002 | Alarm Signal Activity | Well covered by SIA/CID; here for interop |
| 4230003 | **Alarm Disposition Activity** | **No machine-readable standard exists** |
| 4230004 | **Patrol Activity** | **No standard exists at all** |
| 4230005 | **Post Staffing Activity** | **No standard exists at all** |
| 4230006 | **Response Activity** | **No standard exists at all** |
| 4230007 | Incident Report Activity | Partial — EIDD pattern adopted |

**Nine objects:** `portal`, `credential`, `zone`, `alarm_point`, `post`, `checkpoint`, `patrol_tour`, `response_resource`, `disposition`, `signal_code`. Core OCSF `location`, `device`, `user`, `ldap_person`, `organization`, and `actor` are reused rather than duplicated; `location` is patched with site/building/floor.

**One profile:** `physical_site`. Being a profile rather than a class attribute means it can be applied to core IAM and Authentication classes, so a badge-in and a network logon at the same site correlate. That is the physical/logical convergence case, and it's the reason to build on OCSF rather than standalone.

## Validate

```bash
pip install "ocsf-validator>=0.2,<0.3"
git clone https://github.com/ocsf/ocsf-schema.git
cp -r physec ocsf-schema/extensions/physec
python -m ocsf_validator ocsf-schema
```

As of this draft: **12/12 tests pass, with no new warnings** against `ocsf-schema` at `1.10.0-dev`.

To compile and browse (requires Python 3.14+):

```bash
pip install ocsf-schema-compiler
ocsf-schema-compiler /path/to/ocsf-schema -e /path/to/physec -b > browser.json
docker run -it --rm -v $PWD:/app/schemas -e SCHEMA_FILE=/app/schemas/browser.json -p 8080:8080 ocsf-server
```

## UID allocation

Extension `uid` **42** is **not yet reserved**. Reservation is a one-row PR to [`extensions.md`](https://github.com/ocsf/ocsf-schema/blob/main/extensions.md) in `ocsf/ocsf-schema`. 42 was chosen from the unclaimed 4–987 range specifically to keep `type_uid` values inside signed 32-bit range (max here is 423000799); the vendor cluster at 988–999 would produce ~9.9e9 values that overflow any downstream store typed as int32.

Observable `type_id` values are **not** scoped by OCSF, so collisions are real. This extension uses 42001–42008.

## Scope and roadmap

**Committed for v0.1, before publication:**

- ✅ `incident_type_common` ships with a published controlled vocabulary — [`vocabularies/incident_type_common.md`](vocabularies/incident_type_common.md) (24 values, machine-readable [JSON](vocabularies/incident_type_common.json), derived from a live production taxonomy). Crosswalk columns to APCO 2.103.2, NIBRS, and the NFIRS 700-series are structurally present and marked unverified. It is not a free string.
- The SIA DC-03 and APCO 2.103.2 normative texts are paywalled. The crosswalks in this draft were assembled from vendor implementations and will be verified against the purchased standards before any crosswalk is published as authoritative.
- ✅ Continuous validation: every push runs the OCSF validator against the current upstream ocsf-schema (see .github/workflows/validate.yml); 12/12 checks green as of 2026-08-18.

**Committed direction for v0.2:**

- `patrol_tour` generalizes to a recurring-duty model (`site task → occurrence → session → log`) that covers non-patrol recurring duties — equipment checks, lock-ups, escorts — with the same shape. "Tour" is kept in v0.1 for legibility and becomes a specialization of the general form, not a breaking change.

**Deferred:**

- Occupancy and mustering have no interoperable model anywhere. Deferred deliberately until one is worth standardizing.

**Out of scope permanently:**

- Biometric samples must not be carried. Identity references should be tokenized. GDPR Art. 9 and Illinois BIPA both treat biometric identifiers as a special category, and the EU AI Act classifies several biometric uses as high risk.

## License

Apache-2.0 — see [LICENSE](LICENSE).

## References

ONVIF Access Control / Door Control / Credential / Profile M specifications · SIA DC-03-2017, DC-05-2016, ANSI/SIA DC-09-2026, ANSI/SIA OSIPS-01:2008 · SIA OSDP v2.2.2 / IEC 60839-11-5:2020 · ANSI/TMA AVS-01-2024 · ANSI/TMA CS-V-01-2022 · OASIS CAP v1.2 · APCO ANS 2.103.2-2019 · APCO/NENA ANS 2.105.1-2017 (EIDD) · PSIA Common Metadata & Event Model v3.1 · Verizon VERIS
