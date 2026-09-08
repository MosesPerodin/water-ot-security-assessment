# NIST 800-53 / IEC 62443 Crosswalk
## Shenandoah Valley Water Authority

**Purpose:** demonstrate that the segmentation and hardening work documented in Artifact 3 is grounded in recognized federal and industrial-automation security standards, not ad hoc technical choices.
**NIST reference:** SP 800-53 Rev. 5 control catalog
**IEC reference:** ISA/IEC 62443-3-3 System Security Requirements and Security Levels, mapped to the 62443 Foundational Requirements (FR1–FR7)
**Evidence basis:** Artifact 3, RRA, ERP, EPA Checklist Assessment

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

| NIST 800-53 Control | IEC 62443 Requirement | Shenandoah Valley Status | Evidence |
|---|---|---|---|
| **SC-7** (Boundary Protection) | FR5 — Restricted Data Flow; Zone/Conduit segmentation | **Implemented.** Default-deny firewall policy enforced at each of four Purdue-aligned zones (Field/Control, Supervisory, Engineering, Monitoring) | Artifact 3, Sections 5–7 (before/after scan comparison); PLC exposure 5 ports → 1, historian database port closed to Engineering |
| **AC-4** (Information Flow Enforcement) | FR5 — Restricted Data Flow; explicit conduit authorization | **Implemented.** Only two cross-zone conduits authorized (HMI↔PLC Modbus/502, HMI↔Historian PostgreSQL/5432); no other paths permitted between zones | Artifact 3, Section 5 (Segmentation Design); conduit connectivity tests confirming both authorized paths function and no others do |
| **AC-3** (Access Enforcement) | FR2 — Use Control | **Partially implemented.** Network-layer access enforcement in place (SC-7/AC-4 above); host- and application-level access enforcement (e.g., OpenPLC dashboard authorization) not yet addressed | EPA Checklist 2.F (✅); EPA Checklist 2.A (❌ — default credentials still grant full access once network path exists) |
| **IA-5** (Authenticator Management) | FR1 — Identification and Authentication Control | **Implemented (Aug 27, 2026).** Default vendor credentials on the PLC management interface were identified and replaced; old credentials verified dead, new credentials verified working | Artifact 3, Finding 1 (remediated); RRA Section 3/4; EPA Checklist 2.A (now ✅) |
| **IA-2** (Identification and Authentication — incl. MFA) | FR1 — Identification and Authentication Control | **Not implemented.** SSH key-based authentication is in use (stronger than password-only) but does not constitute multi-factor authentication | EPA Checklist 2.H |
| **CM-8** (System Component Inventory) | Asset inventory practice (IEC 62443-2-1 asset management) | **Implemented, with a maturity gap.** Full, accurate host/service/port inventory exists and is evidenced | RRA Section 1 (Asset Inventory); AWWA Assessment Section 1 notes this is point-in-time, not yet continuously maintained |
| **CM-6** (Configuration Settings) | General security configuration management | **Implemented, with a documented historical gap.** Current configuration is correct and live-verified; the assessment specifically found and corrected a case where configuration existed but was not enforced | Artifact 3, Finding 3 and Lessons Learned (configuration-vs-enforcement gap on PLC/Historian firewalls) |
| **SC-8** (Transmission Confidentiality and Integrity) | FR4 — Data Confidentiality | **Not implemented.** HMI web interface transmits over unencrypted HTTP | Artifact 3, Finding 4; EPA Checklist 2.K |
| **IR-8** (Incident Response Plan) | FR6 — Timely Response to Events | **Implemented, not yet exercised.** Written IR plan exists with classification, detection, escalation, containment, and recovery procedures specific to this OT environment | ERP (full document); EPA Checklist 2.S (⚠️ — written but not yet drilled) |
| **IR-4** (Incident Handling) | FR6 — Timely Response to Events | **Implemented, not yet exercised.** Containment procedures explicitly ordered by disruption level; recovery sequencing accounts for conduit dependency (HMI integrity verified before PLC/Historian reconnection) | ERP Sections 4–5 |
| **RA-5** (Vulnerability Scanning / Assessment) | General security assessment practice | **Implemented as a project; not yet recurring.** Comprehensive before/after vulnerability and exposure assessment performed with documented, repeatable methodology | Artifact 3 (full document); RRA Priority 6 recommends this become a scheduled quarterly practice, not yet operationalized |
| **CP-9** (System Backup) | FR7 — Resource Availability | **Not implemented.** No backup schedule or tested recovery capability established for OT configuration, PLC logic, or historian data | EPA Checklist 2.R |
| **SR-2** (Supply Chain Risk Management Plan, family-level) | General supply chain risk practice | **Not implemented.** No vendor risk assessment or upstream vulnerability tracking process exists for OpenPLC or other third-party components in use | AWWA Assessment Section 5 (scored 1, Ad hoc); EPA Checklist 1.G/H |

---

## Summary

Of the twelve controls mapped, **six are fully implemented and evidenced** (SC-7, AC-4, CM-8, CM-6, IR-8/IR-4, IA-5), **one is partially implemented** (AC-3), and **five remain unaddressed** (IA-2, SC-8, CP-9, SR-2).

*(Updated Aug 27, 2026 — IA-5 moved from "Not implemented" to "Implemented" following remediation of Finding 1, the OpenPLC default-credential vulnerability.)*

This distribution is consistent across all Phase 4 documents rather than an artifact of how this crosswalk alone was scored: the implemented controls now cluster around network segmentation, information flow enforcement, configuration management, and — as of this update — credential management, closing the two most directly exploitable gaps identified in the assessment. The unaddressed controls cluster around multi-factor authentication, encryption in transit, backup/recovery, and supply chain — none of which network segmentation resolved on its own, and all of which are already captured in the RRA's prioritized remediation roadmap.

**The intended reading of this crosswalk is not "compliant" or "non-compliant."** IEC 62443 and NIST 800-53 are both control catalogs, not pass/fail certifications, and a crosswalk's value is in showing an assessor's work is traceable to recognized standards — which each row here demonstrates with a specific evidence citation rather than an assertion. The next-highest-value work, per this crosswalk, is IA-2 (MFA), which remains fully open and is a Priority item in EPA's own Top Cyber Actions list.
