# OT/ICS Cybersecurity Assessment Portfolio — Water Sector

A complete, evidence-backed security assessment of a water utility's operational
technology (OT) network, conducted end-to-end in a purpose-built Purdue-aligned
lab. The portfolio covers asset inventory and criticality ranking, network
segmentation before/after remediation, threat detection deployment, a historical
incident mapped to MITRE ATT&CK for ICS, and a full water-sector regulatory
compliance package — presented as an assessor would deliver it to a utility.

**Subject:** Shenandoah Valley Water Authority — a fictional community water
system, modeled on the process and network realities of a real one.

**Author:** Moses Perodin · [LinkedIn](https://www.linkedin.com/in/moses-r-perodin-mba-a78375b/)

**Start here:** [Artifact 3 — Segmentation Assessment](docs/artifacts/Artifact-3-Segmentation-Assessment.md) (flagship, technical) · [Artifact 8 — Client Deliverable Report](docs/artifacts/Artifact-8-Client-Deliverable-Report.md) (executive summary, non-technical)

---

## 1. What This Portfolio Is

This is an **assessment portfolio**, not an engineering demonstration. Every
artifact reads as the work of an assessor establishing whether a control exists
and functions — not as an operator demonstrating exploitation, and not as an
engineer building a SCADA system.

Three things distinguish the work:

- **Consequence-based finding framing.** Findings are ranked by what they let
  someone do to the physical process — interrupt treatment, manipulate a dosing
  setpoint, mask an alarm — not by raw CVSS score. In an OT environment the
  question is not "what can an attacker reach easiest" but "what happens to the
  water if they get there."
- **Operational state over configuration.** Three times in this project, security
  configuration was present in the right files, looked correct on inspection, and
  was doing nothing: firewall rules never enforced by the platform, an IDS
  reporting healthy while inspecting zero packets, a custom rule silently ignored
  on a path mismatch. Every claim traces to a captured command output, and
  re-verification repeats the same commands, flags, and source hosts as the
  original test.
- **Unresolved findings are legitimate output.** Two investigations (a PLC→historian
  egress anomaly, a port-mirroring fault four layers deep) were paused and
  documented rather than chased indefinitely or quietly dropped. Two false-positive
  conclusions were caught mid-assessment and corrected in place rather than
  smoothed over.

---

## 2. How to Navigate the Artifacts

All artifacts live in [`docs/artifacts/`](docs/artifacts/). They are meant to be
read in roughly this order — each depends on inputs from the ones before it.

| # | Artifact | What it establishes |
|---|---|---|
| 2 | [ICS401V Operations Linkage](docs/operations-linkage/ICS401V-Operations-Linkage.md) | The operations-to-assessment reasoning the rest of the portfolio applies |
| 3 | [Segmentation Assessment](docs/artifacts/Artifact-3-Segmentation-Assessment.md) **(flagship)** | Before/after network segmentation, verified by repeat testing with a disclosed methodology change and the HMI's mid-window remediation noted. Origin of Findings 1–4 |
| 4 | [Asset Inventory & Criticality](docs/artifacts/Artifact-4-Asset-Inventory-Criticality.md) | Consequence-based criticality ranking; feeds the RRA's asset register |
| 5 | [MITRE ATT&CK — Oldsmar Case Study](docs/artifacts/Artifact-5-ATTCK-Oldsmar-Case-Study.md) | Blue-team retrospective on the 2021 Oldsmar incident, lessons applied to this architecture |
| 6 | [Threat Detection Assessment](docs/artifacts/Artifact-6-Threat-Detection-Assessment.md) | Suricata deployment, detection-coverage evaluation, positive-control verification. Origin of Findings 6.1–6.4 |
| 6a | [Mirroring Investigation Handoff](docs/artifacts/Artifact-6-Mirroring-Investigation-Handoff.md) | Linked but separate: a paused port-mirroring investigation, using `Fault N` vocabulary by design (see §3) |
| 7 | [Water Sector Cyber Risk Package](docs/artifacts/Artifact-7-Water-Sector-Cyber-Risk-Package.md) **(flagship)** | Integration layer for the compliance package. Origin of Findings 7.1–7.3 |
| 8 | [Client Deliverable Report](docs/artifacts/Artifact-8-Client-Deliverable-Report.md) | Executive synthesis for a non-technical reader; findings re-presented A–H by operational consequence |

Artifact 7 is an **integration document** spanning six files — each serves different regulatory purposes and audiences, revised on different cadences. See [`docs/artifacts/Artifact-7-Water-Sector-Cyber-Risk-Package.md`](docs/artifacts/Artifact-7-Water-Sector-Cyber-Risk-Package.md) for full component breakdown.

### Frameworks in scope

| Framework | Role |
|---|---|
| IEC 62443 | Zone/conduit model, security levels, crosswalk target |
| NIST SP 800-82 | OT-specific control guidance and rationale |
| NIST SP 800-53 | Control catalogue, crosswalk source |
| Purdue Model | Zone framing for segmentation work |
| MITRE ATT&CK for ICS | Technique mapping, case-study structure |
| AWIA §2013 | Risk and resilience assessment driver |
| EPA 817-B-23-001 | Water-sector cybersecurity checklist |
| AWWA assessment framework | Sector self-assessment |
| NIST AI RMF 1.0 | Destination framework (Phase 6, post-MVP) |

---

## 3. Finding Identifier Convention

The portfolio uses **artifact-scoped finding identifiers**: a finding is numbered
by the artifact that identified it.

| Scheme | Meaning |
|---|---|
| `Finding N` (bare integer) | Findings 1–4, cross-referenced by ID across eight committed documents; not renumbered |
| `Finding N.N` (artifact-scoped) | 3.1, 6.1–6.4, 7.1–7.3 — findings surfaced during or after remediation |
| `Finding A`–`H` (lettered) | Audience-specific re-presentation for non-technical readers (Artifact 8 only) |
| `Fault N` | Unresolved debugging observations, not security findings (Artifact 6a only) |

Authoritative status: [`docs/project-knowledge/FINDINGS-REGISTER.md`](docs/project-knowledge/FINDINGS-REGISTER.md).

**Open findings (High):** 7.1 (No MFA), 7.2 (No backup/recovery), 7.3 (No OT accountability). **Open findings (Medium):** 4 (HMI unencrypted), 6.3 (Sensor behind firewall), 6.4 (No rate-based SSH detection).

---

## 4. Credentials

- **CompTIA Security+** (June 2025)
- **CISA ICS300** (86%, July 2026)
- **CISA ICS401V** — all 13 modules (July 2026)

Certificates: [`docs/training-certificates/`](docs/training-certificates/)

---

## 5. License

[MIT License](LICENSE). The subject utility, its network, and all findings are
fictional and were produced in an isolated lab.
