# OT/ICS Cybersecurity Assessment Portfolio — Water Sector

A complete, evidence-backed security assessment of a water utility's operational
technology (OT) network, conducted end-to-end in a purpose-built Purdue-aligned
lab. The portfolio covers asset inventory and criticality ranking, network
segmentation before/after remediation, threat detection deployment, a historical
incident mapped to MITRE ATT&CK for ICS, and a full water-sector regulatory
compliance package — presented as an assessor would deliver it to a utility.

**Subject:** Shenandoah Valley Water Authority — a fictional community water
system, modeled on the process and network realities of a real one.

**Author:** Moses Perodin — wastewater operations professional with 10 years plant
management experience; worked with Ignition SCADA HMI for monitoring and
operational insight; now pivoting to OT/ICS cybersecurity assessment.

**Start here:** [Artifact 3 — Segmentation Assessment](docs/artifacts/Artifact-3-Segmentation-Assessment.md) (flagship, technical) · [Artifact 8 — Client Deliverable Report](docs/artifacts/Artifact-8-Client-Deliverable-Report.md) (executive summary, non-technical)

**Connect:** [LinkedIn](https://www.linkedin.com/in/moses-r-perodin-mba-a78375b/)

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
  re-verification uses the identical method as the original test.
- **Unresolved findings are legitimate output.** Two investigations (a PLC→historian
  egress anomaly, a port-mirroring fault four layers deep) were paused and
  documented rather than chased indefinitely or quietly dropped. Two false-positive
  conclusions were caught mid-assessment and corrected in place rather than
  smoothed over.

### The operations-to-assessment angle

The competitive premise of this portfolio is that an operator ranks OT assets by
**consequence to the physical process**, while a pure-IT assessor ranks them by
**network exposure** — and in an OT environment those two orderings produce
materially different, and better, prioritization.

That premise is worked out concretely in
[`docs/operations-linkage/ICS401V-Operations-Linkage.md`](docs/operations-linkage/ICS401V-Operations-Linkage.md),
which maps CISA ICS401V assessment concepts to direct experience running a
pretreatment anaerobic digester plant:

| Operational experience | Assessment concept it informs |
|---|---|
| Biogas flare as an explosion barrier | Safety Instrumented System (SIS) integrity as a separate, higher-priority category (TRITON/TRISIS) |
| DAF chemical dosing drift cascading to a downstream plant's permit | Silent setpoint manipulation as a stealth attack vector (Stuxnet) |
| Buffered vs. unbuffered process stages | Criticality ranking by operational impact, not generic CVSS |
| SBR phase-dependent instrument readings | Process-state-aware anomaly detection, not just protocol signatures |
| A sequential, functionally isolated treatment train | Whether network segmentation actually matches process segmentation |

This linkage runs through the whole artifact set: the PLC is the RRA's highest
risk because it is the only asset with a direct, unmediated path to physical
consequence — an operations judgment, not a scan result.

---

## 2. The Lab

A virtualized OT/ICS testbed on Proxmox VE, segmented into Purdue-aligned zones.
The lab is a *representative environment*, not a claim of a real utility network;
where it diverges from a real plant, the artifacts say so.

| Zone (bridge) | Subnet | Hosts |
|---|---|---|
| Field / Control (`vmbr10`) | 192.168.10.0/24 | VM101 `plc-controller` — OpenPLC Runtime |
| Supervisory / HMI (`vmbr20`) | 192.168.20.0/24 | VM100 `hmi-scada` (Apache) · VM102 `historian-db` (PostgreSQL) |
| Engineering (`vmbr30`) | 192.168.30.0/24 | VM103 `workstation-eng` · VM105 `kali-attack` (assessment platform) |
| Monitoring (`vmbr40`) | 192.168.40.0/24 | VM104 `monitor-wireshark` — Suricata 7.0.3 IDS |
| Management (`vmbr0`) | 192.168.1.0/24 | Proxmox host |

Legitimate operational conduits: HMI↔PLC (Modbus TCP/502), HMI↔Historian
(PostgreSQL/5432), all zones→Monitoring (TCP/514). Everything else is default-deny.
The full layout, with a Purdue-level overlay and the authorized-conduit table, is
diagrammed in
[`docs/diagrams/Zone-Conduit-Purdue-Overlay.md`](docs/diagrams/Zone-Conduit-Purdue-Overlay.md).

Lab build history is in [`docs/build-journal.md`](docs/build-journal.md);
live topology (generated read-only from the hypervisor) is in
[`docs/project-knowledge/LAB-TOPOLOGY.md`](docs/project-knowledge/LAB-TOPOLOGY.md).

---

## 3. How to Navigate the Artifacts

All artifacts live in [`docs/artifacts/`](docs/artifacts/). They are meant to be
read in roughly this order — each depends on inputs from the ones before it.

| # | Artifact | What it establishes |
|---|---|---|
| 2 | [ICS401V Operations Linkage](docs/operations-linkage/ICS401V-Operations-Linkage.md) | The operations-to-assessment reasoning the rest of the portfolio applies |
| 3 | [Segmentation Assessment](docs/artifacts/Artifact-3-Segmentation-Assessment.md) **(flagship)** | Before/after network segmentation, verified by identical repeat testing. Origin of Findings 1–4 |
| 4 | [Asset Inventory & Criticality](docs/artifacts/Artifact-4-Asset-Inventory-Criticality.md) | Consequence-based criticality ranking; feeds the RRA's asset register |
| 5 | [MITRE ATT&CK — Oldsmar Case Study](docs/artifacts/Artifact-5-ATTCK-Oldsmar-Case-Study.md) | Blue-team retrospective on the 2021 Oldsmar incident, lessons applied to this architecture |
| 6 | [Threat Detection Assessment](docs/artifacts/Artifact-6-Threat-Detection-Assessment.md) | Suricata deployment, detection-coverage evaluation, positive-control verification. Origin of Findings 6.1–6.4 |
| 6a | [Mirroring Investigation Handoff](docs/artifacts/Artifact-6-Mirroring-Investigation-Handoff.md) | Linked but separate: a paused port-mirroring investigation, using `Fault N` vocabulary by design (see §4) |
| 7 | [Water Sector Cyber Risk Package](docs/artifacts/Artifact-7-Water-Sector-Cyber-Risk-Package.md) **(flagship)** | Integration layer for the compliance package. Origin of Findings 7.1–7.3 |
| 8 | [Client Deliverable Report](docs/artifacts/Artifact-8-Client-Deliverable-Report.md) | Executive synthesis for a non-technical reader; findings re-presented A–H by operational consequence |

### Why Artifact 7 is one artifact in six files

Artifact 7 is an **integration document**. It exists because its five component
deliverables are not independent — each one's conclusions depend on inputs from
the others, and reading any one in isolation gives an incomplete picture. They
are separate files because they serve different regulatory purposes, are read by
different audiences, and are revised on different cadences (a statutory RRA on a
five-year cycle; an ERP whenever the architecture changes). Merging them would
make each worse.

| File | Role |
|---|---|
| [`Artifact-7-Water-Sector-Cyber-Risk-Package.md`](docs/artifacts/Artifact-7-Water-Sector-Cyber-Risk-Package.md) | The connective tissue — how the components interlock and in what order they must be updated |
| [`RRA-Shenandoah-Valley-Water-Authority.md`](docs/artifacts/RRA-Shenandoah-Valley-Water-Authority.md) | Risk & Resilience Assessment — *statutorily required* (AWIA §2013) |
| [`ERP-Shenandoah-Valley-Water-Authority.md`](docs/artifacts/ERP-Shenandoah-Valley-Water-Authority.md) | Emergency Response Plan — *statutorily required* (AWIA §2013) |
| [`AWWA-Assessment-Shenandoah-Valley-Water-Authority.md`](docs/artifacts/AWWA-Assessment-Shenandoah-Valley-Water-Authority.md) | AWWA maturity self-assessment — voluntary |
| [`EPA-Checklist-Shenandoah-Valley-Water-Authority.md`](docs/artifacts/EPA-Checklist-Shenandoah-Valley-Water-Authority.md) | EPA 817-B-23-001 item-level gap analysis — voluntary |
| [`Crosswalk-NIST80053-IEC62443-Shenandoah-Valley.md`](docs/artifacts/Crosswalk-NIST80053-IEC62443-Shenandoah-Valley.md) | Control-level traceability to NIST 800-53 Rev. 5 and ISA/IEC 62443-3-3 |

### Supporting material

- [`docs/case-studies/001-network-segmentation/`](docs/case-studies/001-network-segmentation/) — the segmentation work as a standalone case study, with the raw connectivity-test and firewall-counter evidence under `evidence/`
- [`configs/firewall/network-segmentation/`](configs/firewall/network-segmentation/) — the Proxmox firewall policies as deployed
- [`docs/troubleshooting/`](docs/troubleshooting/) — Proxmox networking lessons learned
- [`docs/project-knowledge/`](docs/project-knowledge/) — methodology, findings register, portfolio state, lab topology

### Frameworks in scope

| Framework | Role in the portfolio |
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

Control identifiers are verified against primary sources before they enter a document — see [`docs/project-knowledge/ASSESSMENT-METHODOLOGY.md`](docs/project-knowledge/ASSESSMENT-METHODOLOGY.md) for the full methodology this portfolio follows.

---

## 4. Finding Identifier Convention

The portfolio uses **artifact-scoped finding identifiers**: a finding is numbered
by the artifact that identified it.

| Scheme | Used by | Meaning |
|---|---|---|
| `Finding N` (bare integer) | Artifact 3 | Findings 1–4. Predate the scoped convention; **not renumbered**, because they are cross-referenced by ID across eight committed documents |
| `Finding N.N` (artifact-scoped) | Artifacts 3, 6, and 7 | 3.1 (Artifact 3 follow-up — sub-finding surfaced during remediation, after Findings 1–4 were already fixed as bare integers; scoped notation marks it as a distinct, later-discovered issue rather than triggering renumbering), 6.1–6.4 (detection coverage), 7.1–7.3 (systemic/governance) |
| `Finding A`–`H` (lettered) | Artifact 8 | An audience-specific re-presentation of the above for a non-technical reader — *not* a separate investigation. Every letter traces back to a numbered finding |
| `Fault N` | Mirroring Investigation Handoff (6a) | Deliberately **not** called a "finding" — these are unresolved debugging observations, not security findings against the assessed environment |

Authoritative status for every finding, and every place it is referenced, lives
in [`docs/project-knowledge/FINDINGS-REGISTER.md`](docs/project-knowledge/FINDINGS-REGISTER.md).
Summary:

| ID | Finding | Status |
|---|---|---|
| 1 | Default credentials on the PLC web management interface | ✅ Remediated (Aug 27, 2026) |
| 2 | Historian database access controlled at the application layer only (High) | ✅ Remediated (Aug 26, 2026) |
| 3 | Configuration-vs-enforcement gap on PLC and historian firewalls (Medium) | ✅ Remediated (Aug 26, 2026) |
| 3.1 | PLC-to-historian egress anomaly — root cause identified (Proxmox `firewall=1` + `br_netfilter` NAT interaction) | ⚠️ Root cause identified; symptom cleared by host reboot but underlying condition persists |
| 4 | HMI web interface unencrypted (HTTP only, no TLS) (Medium) | ⬜ Open |
| 6.1 | Stock Suricata config referenced a nonexistent interface — 14,000+ silent restart cycles | ✅ Remediated |
| 6.2 | Custom detection rule silently not loaded — rule-path resolution mismatch | ✅ Remediated |
| 6.3 | Sensor behind default-deny firewall structurally cannot see blocked traffic | ⬜ Open |
| 6.4 | Stock ET Open ruleset has no rate-based SSH brute-force detection | ⬜ Open |
| 7.1 | No multi-factor authentication anywhere in the OT environment | ⬜ Open (High) |
| 7.2 | No backup or tested recovery capability for OT systems | ⬜ Open (High) — hypervisor-level confirmation: [`docs/evidence/backup-posture-20260901.txt`](docs/evidence/backup-posture-20260901.txt) |
| 7.3 | No named OT cybersecurity accountability or change control | ⬜ Open (High) |

---

## 5. Methodology Principles

Established during the work and carried across every artifact. Full detail in
[`docs/project-knowledge/ASSESSMENT-METHODOLOGY.md`](docs/project-knowledge/ASSESSMENT-METHODOLOGY.md).

- **Configuration ≠ enforcement.** Verify operational state — rule counts,
  enforcement counters, restart counts — not the presence of a config file.
- **Verify the pipeline before interpreting an absence of detections.** "No alerts
  fired" is equally consistent with good coverage, a coverage gap, and a broken
  sensor. The positive-control test is what tells them apart.
- **Consequence-based ranking produces different priorities than exploitability-based
  ranking.** This is the operations background doing analytical work, not framing.
- **Restraint is a deliverable.** `-T3` scan timing is deliberate — aggressive
  scanning can destabilize legacy PLCs. An assessment that disrupts the process it
  protects has failed regardless of findings. The restraint is documented.
- **Phase order is strict:** capture the complete before-evidence set → baseline
  services → implement the change → repeat identical tests → compare. No changes
  until the full before-set is captured.

---

## 6. Credentials

- **CompTIA Security+** (June 2025)
- **CISA ICS300** (86%, July 2026)
- **CISA ICS401V** — all 13 modules (July 2026)

Certificates: [`docs/training-certificates/`](docs/training-certificates/)

---

## 7. Education

- **MBA** — Georgia Southern University
- **BS, Recreation and Sports Management** — Florida International University

---

## 8. Roadmap

The assessment scope above is complete. The portfolio's next phase extends the
same assessment discipline to AI risk in critical infrastructure — a NIST AI RMF
assessment of ML-based chemical dosing at the same fictional utility, crosswalked
to IEC 62443 and NIST SP 800-82.

---

## 9. License

[MIT License](LICENSE). The subject utility, its network, and all findings are
fictional and were produced in an isolated lab.
