# Water Sector Cyber Risk Package
## Shenandoah Valley Water Authority — Regulatory Compliance and Risk Governance Documentation

**Artifact type:** Integrated compliance package (five component documents)
**Regulatory basis:** America's Water Infrastructure Act of 2018, Section 2013 (amending Safe Drinking Water Act §1433)
**Supporting frameworks:** EPA 817-B-23-001, AWWA cybersecurity guidance, NIST SP 800-53 Rev. 5, ISA/IEC 62443-3-3, NIST SP 800-82
**Assessment period:** August 2026

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## 1. Purpose and Structure

This artifact is the integration layer for a five-document compliance package. It exists because the package is not five independent deliverables that happen to share a subject — the documents depend on each other, and reading any one in isolation gives an incomplete and in places misleading picture.

**Why this is one artifact in five files rather than a single document:** the components serve different regulatory and operational purposes, are read by different audiences, and are updated on different cadences. A Risk and Resilience Assessment is a statutorily-required artifact with a five-year review cycle. An Emergency Response Plan is an operational document that responders use during an incident and that should be revised whenever the architecture changes. Merging them would make both worse. This document is the connective tissue.

### Component documents

| Component | Purpose | Primary audience | Regulatory anchor |
|---|---|---|---|
| **Risk & Resilience Assessment** | Identify assets, threats, vulnerabilities; quantify and prioritize risk | Utility leadership, board | AWIA §2013 requirement |
| **Emergency Response Plan** | Detect, classify, contain, and recover from incidents | Operations staff, on-call responders | AWIA §2013 requirement |
| **AWWA Self-Assessment** | Score security maturity across control domains | Leadership, board, planning | Voluntary sector framework |
| **EPA Cybersecurity Checklist** | Item-level gap analysis against federal guidance | Security staff, auditors | EPA 817-B-23-001 (voluntary tool) |
| **NIST 800-53 / IEC 62443 Crosswalk** | Map implemented controls to recognized standards | Technical assessors, auditors | Control traceability |

---

## 2. Regulatory Context

Section 2013 of the America's Water Infrastructure Act of 2018 amended Safe Drinking Water Act §1433 to require community water systems serving more than 3,300 people to conduct a Risk and Resilience Assessment and to prepare or revise an Emergency Response Plan. Both must address malevolent acts and natural hazards, and both carry a recurring review obligation on a five-year cycle.

**Two points about scope that matter for how this package should be read:**

**First, AWIA's requirements are not exclusively cybersecurity requirements.** The statute covers physical security, natural hazards, and operational resilience alongside cyber risk. This package addresses the cybersecurity dimension. A complete AWIA submission would require physical security and natural hazard assessment that is explicitly outside this engagement's scope (see Section 5).

**Second, only two of the five components are required.** The RRA and ERP satisfy statutory obligations. The AWWA self-assessment, EPA checklist, and standards crosswalk are voluntary tools. They are included because a compliance artifact that satisfies a requirement without improving security has missed the point — the voluntary components are what turn the required documents from a filing exercise into an actual gap analysis with a prioritized remediation path.

---

## 3. How the Components Interlock

The package was built in a deliberate order, and each document's conclusions depend on inputs from the ones before it.

**Asset inventory and criticality ranking** (Artifact 4) → feeds the RRA's asset register and directly determines its risk weighting. The RRA scores "unauthorized PLC parameter modification" as its highest risk specifically because the criticality analysis established the PLC as the only asset with a direct, unmediated path to physical process consequence.

**Segmentation assessment evidence** (Artifact 3) → provides the RRA's vulnerability inputs and the EPA checklist's implemented-control evidence. Findings 1 through 4 originate there and are cited, not re-derived, in every downstream document.

**RRA risk matrix and residual risk analysis** → determines ERP incident severity tiers. The ERP classifies unauthorized PLC access as Severity 1 because the RRA scored the corresponding risk highest, not as an independent judgment.

**RRA residual risk finding** (the HMI became a concentrated trust point post-segmentation) → directly produces the ERP's recovery sequencing requirement that HMI integrity be verified before either the PLC or historian is reconnected. This is the clearest example of the package's interdependence: an analytical finding in one document becomes a mandatory operational step in another.

**AWWA maturity scores and EPA checklist gaps** → surface the systemic weaknesses that individual technical findings do not, producing Findings 7.1 through 7.3 below.

**Crosswalk** → provides control-level traceability across all of the above, mapping each implemented control to NIST 800-53 and IEC 62443 with a specific evidence citation rather than an assertion.

**The practical implication:** if the architecture changes, the update order matters. Asset inventory first, then segmentation evidence, then RRA, then ERP. Revising the ERP without revisiting the RRA that determined its severity tiers would produce an internally inconsistent package.

---

## 4. Findings Identified by This Package

These three findings emerged from the compliance analysis rather than from technical testing. They are recorded here as numbered findings because they are among the highest-priority open items in the entire assessment, and items that exist only inside a summary document tend not to get tracked.

### Finding 7.1 — No multi-factor authentication anywhere in the OT environment *(Open, High priority)*

**Source:** EPA Checklist item 2.H (a designated priority action in joint EPA/CISA/FBI guidance); NIST 800-53 IA-2; IEC 62443 FR1.

**Current state:** remote administrative access uses cryptographic key-based authentication, which is meaningfully stronger than passwords alone. It is not multi-factor. No OT system in the environment requires a second authentication factor.

**Consequence framing:** every remaining access control in this environment rests on a single secret. Finding 1 (default PLC credentials) demonstrated concretely what happens when one such secret is weak — it granted full administrative access to the process controller with no exploit required. MFA is the control that limits damage when a credential is compromised rather than assuming none ever will be. Given that credential weakness was the highest-severity technical finding in this assessment, the absence of a compensating control against credential compromise is the most significant remaining gap.

**Why this was not caught by technical testing:** scanning and segmentation testing establish what is reachable. They do not establish what authentication is required once something is reached. This finding is a product of framework-driven gap analysis specifically, which is the argument for including the voluntary components in this package.

### Finding 7.2 — No backup or tested recovery capability for OT systems *(Open, High priority)*

**Source:** EPA Checklist item 2.R; NIST 800-53 CP-9; IEC 62443 FR7 (Resource Availability).

**Current state:** no backup schedule exists for control logic, OT system configuration, or historian data. No recovery has been tested.

**Consequence framing:** the Emergency Response Plan in this package defines a specific, dependency-ordered recovery sequence. That plan presumes there is something to recover from. Without tested backups, recovery from a destructive incident, ransomware event, or significant hardware failure would mean reconstructing control logic and configuration from institutional memory — during an active incident, under time pressure, with the treatment process affected.

**The distinction between "backups exist" and "recovery has been tested" is deliberate.** Untested backups are a common failure mode: the backup job runs, reports success, and the restoration is discovered to be incomplete or corrupt only when it is needed. The recommendation is a tested recovery, not a configured backup.

### Finding 7.3 — No named accountability or change control for OT cybersecurity *(Open, High priority)*

**Source:** EPA Checklist items 1.B and 1.C; AWWA Governance & Asset Management maturity score (2 — Repeatable).

**Current state:** no individual holds named responsibility for OT cybersecurity. No change-control process governs the addition of services, accounts, or credentials on OT systems. No reassessment cadence is established.

**Consequence framing:** this is the condition that allowed Findings 1 through 4 to persist undetected. The default credential in Finding 1 was not the result of a difficult technical problem — it was the result of no process existing that would have caught it. Similarly, the configuration-versus-enforcement gap in Finding 3 (firewall rules present in configuration files but never enforced by the platform) went unnoticed because no one was responsible for verifying that configured controls were operating.

**Why this ranks alongside the technical findings rather than below them:** every technical remediation documented in this portfolio is a point-in-time correction. Firewall rules drift, staff turn over, new equipment arrives carrying new vendor defaults. Without ownership and change control, the improvements verified in August 2026 have no mechanism keeping them true in August 2027. This finding is what determines whether the rest of the assessment has durable value or produces a secure snapshot.

**Effort note:** unlike 7.1 and 7.2, this finding requires no budget and no technical work. It requires a decision about who owns it.

---

## 5. What This Package Does Not Cover

**Physical security.** AWIA §2013 requires assessment of physical barriers, monitoring practices, and physical access controls. This package addresses cybersecurity only. The AWWA self-assessment leaves its Physical Security section explicitly unscored rather than assigning a placeholder value, because the assessed environment was virtualized and produces no meaningful physical security evidence.

**Natural hazards.** AWIA requires assessment of resilience to natural hazards. Out of scope here.

**Financial and operational resilience.** AWIA's scope includes the resilience of financial infrastructure and operational continuity beyond cyber incidents.

**Personnel and training.** Security awareness training and OT-specific role training are EPA checklist items (2.I, 2.J) marked not applicable to this assessment's single-operator scope. A real utility would need to address them.

**IT network and IT/OT boundary.** Scope was limited to the OT environment. The boundary between business systems and process control is a well-documented attack path in the water sector and warrants separate assessment.

**A complete AWIA submission would require all of the above.** This package should be understood as the cybersecurity component of a compliance obligation, not as satisfying that obligation in full.

---

## 6. What the Package Collectively Establishes

Read together, the five components support four conclusions that no single document supports on its own:

**The highest-consequence technical gaps are closed and verified closed.** Segmentation and credential remediation are evidenced by before/after testing (Artifact 3), mapped to specific controls (crosswalk: SC-7, AC-4, IA-5), and reflected consistently across the EPA checklist and AWWA scoring.

**The remaining gaps cluster in a specific and predictable area.** Across three independently-scored frameworks, the same pattern appears: network and configuration controls score well; identity, recovery, and governance controls score poorly. The AWWA scores (Network 3, Access Control 2, Governance 2, Supply Chain 1), the EPA checklist distribution, and the crosswalk's implemented-versus-unaddressed split all converge on the same conclusion. That convergence is itself evidence that the finding is real and not an artifact of one scoring method.

**Improvement is narrow and project-based, not organizational.** Every framework in this package independently caps its assessment of the Authority's maturity below "managed" for the same reason: what exists is a well-executed one-time engagement with a recommended re-verification cadence that has not yet run a second time. Findings 7.1 through 7.3 are the specific gaps between where the Authority is and a maintained practice.

**The documents are internally consistent and cross-verified.** Finding statuses, control identifiers, and evidence citations were reconciled across all components before commit. Where a status changed during the engagement (Finding 1's remediation on August 27), every referencing document was updated, and a subsequent audit caught and corrected one remaining inconsistency. That reconciliation process is documented rather than assumed.

---

## 7. Maintenance Requirements

| Trigger | Required action |
|---|---|
| Five-year statutory cycle | RRA review and recertification; ERP revision within six months |
| Architecture change (new zone, conduit, or asset) | Update asset inventory → segmentation evidence → RRA → ERP, in that order |
| Finding status change | Update every referencing document; reconcile before committing |
| Quarterly (recommended, not yet operating) | Re-scan against documented baseline; compare to Artifact 3 after-state |
| New OT system or service deployed | Asset inventory update; credential and configuration review before production |
| Post-incident | ERP effectiveness review; RRA risk re-scoring if the incident revealed unassessed risk |

The quarterly re-verification is the item that moves segmentation maturity from Defined (3) to Managed (4). It is currently a recommendation with no owner, which makes it a specific instance of Finding 7.3.

---

## 8. Component Document Index

| File | Component |
|---|---|
| `RRA-Shenandoah-Valley-Water-Authority.md` | Risk & Resilience Assessment |
| `ERP-Shenandoah-Valley-Water-Authority.md` | Emergency Response Plan |
| `AWWA-Assessment-Shenandoah-Valley-Water-Authority.md` | AWWA Cybersecurity Self-Assessment |
| `EPA-Checklist-Shenandoah-Valley-Water-Authority.md` | EPA Cybersecurity Checklist Assessment |
| `Crosswalk-NIST80053-IEC62443-Shenandoah-Valley.md` | NIST 800-53 / IEC 62443 Crosswalk |

**Related artifacts outside this package:** Artifact 3 (Segmentation Assessment) supplies this package's technical evidence base; Artifact 4 (Asset Inventory & Criticality) supplies its asset register and risk weighting; Artifact 8 (Client Deliverable Report) presents this package's conclusions to a non-technical audience.

---

## 9. Note on Finding Identifier Conventions

This portfolio uses artifact-scoped finding identifiers: a finding is numbered by the artifact that identified it. Artifact 6 identified Findings 6.1–6.4; this artifact identifies Findings 7.1–7.3.

Findings 1 through 4 predate this convention and retain bare integer identifiers from Artifact 3. They are not renumbered, because they are cross-referenced by identifier across eight committed documents and renumbering would break those references for no analytical gain.

The mirroring investigation handoff uses a separate `Fault N` vocabulary deliberately. Those items are unresolved debugging observations, not assessment findings, and using distinct terminology prevents them from being read as security findings against the assessed environment.

Artifact 8 uses lettered findings (A–H) for a non-technical audience. Findings A, C, and D correspond to Findings 1, 2, and 4; Finding B draws on Findings 1 and 3; Finding F consolidates 6.3 and 6.4; and Findings E, G, and H correspond to Findings 7.1, 7.2, and 7.3 as formalized in this document.
