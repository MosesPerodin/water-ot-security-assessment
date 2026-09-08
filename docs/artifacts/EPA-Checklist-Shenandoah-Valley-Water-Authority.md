# EPA Cybersecurity Checklist Assessment
## Shenandoah Valley Water Authority

**Reference:** EPA Cybersecurity Checklist for Drinking Water and Wastewater Systems (EPA 817-B-23-001, Appendix A), derived from CISA Cross-Sector Cybersecurity Performance Goals, aligned to NIST CSF
**Legend:** ✅ Implemented — ⚠️ Partial / gap identified — ❌ Not implemented — N/A Not applicable to this assessment's scope
**Priority items** (marked with EPA's own `*`) are drawn from the joint EPA/CISA/FBI "Top Cyber Actions for Securing Water Systems"
**Evidence basis:** Artifact 3, RRA, ERP, AWWA Assessment

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## 1. IDENTIFY

| # | Does the WWS... | Status | Notes |
|---|---|---|---|
| 1.A* | Maintain an updated inventory of all OT and IT network assets? | ⚠️ | Inventory exists and is accurate (RRA Section 1) but is a point-in-time assessment artifact, not yet a continuously maintained record — AWWA Governance score capped at 2 (Repeatable) for this reason |
| 1.B* | Have a named role responsible for cybersecurity activities? | ❌ | No formally named role/position established for this environment as part of the assessment cycle |
| 1.C | Have a named role responsible for OT-specific cybersecurity? | ❌ | Not established |
| 1.D | Provide regular OT/IT communication opportunities? | N/A | Lab environment does not have a separate IT organization to coordinate with |
| 1.E* | Patch or mitigate known vulnerabilities within a recommended timeframe? | ⚠️ | All hosts run a uniform, current OpenSSH baseline (9.6p1) at assessment time, but no formal, ongoing patch-management process exists |
| 1.G/H | Require OT vendors to notify of security incidents/vulnerabilities? | ❌ | No vendor notification requirements established (AWWA Supply Chain score: 1, Ad hoc) |
| 1.I | Include cybersecurity as a procurement evaluation criterion? | ❌ | No procurement process exists in this lab context |

---

## 2. PROTECT

| # | Does the WWS... | Status | Notes |
|---|---|---|---|
| 2.A* | Change default passwords? | ✅ *(Aug 27, 2026)* | **Finding 1 (Artifact 3) — remediated.** OpenPLC's default vendor credentials (openplc/openplc) were confirmed live via direct HTTP test, then replaced. Old credentials confirmed dead and new credentials confirmed working via the same test method. This had been the single most significant open item in this assessment. |
| 2.B* | Require a minimum password length? | ⚠️ | No formal password policy verified across OT assets |
| 2.C* | Require unique, separate OT vs. IT credentials per user? | ⚠️ | Not formally assessed; single-operator lab environment does not yet exercise this control |
| 2.D* | Immediately disable access when no longer needed? | ⚠️ | No user lifecycle process tested in this assessment cycle |
| 2.E* | Separate user and privileged (administrator) accounts? | ⚠️ | Partial — sudo access was deliberately scoped to specific commands and removed after use during Phase 2 (verified via `sudo -n` failure test), demonstrating the practice, but no formal, standing policy governs it |
| 2.F | Segment OT and IT networks, deny by default? | ✅ | **This is the core achievement of Artifact 3.** Zone-based, default-deny segmentation implemented and independently verified: PLC exposure reduced 5 ports → 1; historian database port closed to Engineering entirely |
| 2.G | Detect and block repeated failed logins? | ❌ | Not implemented |
| 2.H* | Require MFA, at minimum for remote OT/IT access? | ❌ | SSH key-based authentication is in use (stronger than password auth) but does not constitute MFA; no MFA implemented |
| 2.I* | Annual cybersecurity awareness training for all personnel? | N/A | Not applicable to this assessment's single-operator lab scope |
| 2.J | OT-specific annual training for OT personnel? | N/A | Same as above |
| 2.K | Encrypt data in transit (TLS/SSL)? | ❌ | **Finding 4 (Artifact 3):** HMI web interface runs HTTP only, no TLS |
| 2.L | Encrypt stored sensitive data? | ⚠️ | Not independently verified for the historian's data-at-rest; PostgreSQL's default configuration was not audited for this |
| 2.M | Email security controls? | N/A | No email infrastructure in this environment |
| 2.N | Disable Office macros by default? | N/A | No Windows/Office assets in this environment |
| 2.O* | Maintain current documentation of OT/IT asset configuration? | ✅ | Artifact 3 documents exact before/after configuration state for every assessed host, with live-enforcement verification (not just config-file review) — directly addresses the configuration-vs-enforcement gap identified as Finding 3 |
| 2.P | Maintain network topology documentation? | ✅ | Zone/subnet architecture fully documented (RRA Section 1, Artifact 3 throughout) and aligned to the Purdue Model |
| 2.Q | Require approval before new software is installed? | ⚠️ | Sudo scoping during Phase 2 demonstrates the underlying discipline, but no formal approval process exists |
| 2.R* | Backup systems on a regular, tested schedule? | ❌ | Not addressed in this assessment cycle |
| 2.S* | Have a written IR plan, regularly practiced and updated? | ⚠️ | **ERP now exists** with full incident classification, detection triggers, escalation chain, containment, recovery, and communication sections — but it has not yet been exercised via tabletop or drill (AWWA Section 4 flags this explicitly) |
| 2.T | Collect security logs for detection/investigation? | ⚠️ | Monitoring zone and host exist and are correctly segmented, but log collection is not yet comprehensive or centrally verified |
| 2.U | Protect security logs from tampering? | ❌ | Not assessed |
| 2.V | Prohibit unauthorized hardware connections? | N/A | Virtualized lab environment; physical port control does not apply |
| 2.W* | Ensure no unnecessary exploitable services on exposed assets? | ✅ | Directly addressed by segmentation remediation — PLC's four ICS protocol ports (Modbus, EtherNet/IP, S7comm, HTTP management) are no longer reachable from Engineering |
| 2.X* | Eliminate OT asset connections to the public Internet? | ✅ | OT-zone hosts, including the assessment platform, have no internet egress by design (verified architectural constraint) |

---

## 3. DETECT

| # | Does the WWS... | Status | Notes |
|---|---|---|---|
| 3.A | Maintain a list of relevant threats/adversary TTPs? | ⚠️ | The RRA's MITRE ATT&CK for ICS mapping (Section 2) identifies the specific technique chain relevant to this environment, but this is a one-time analysis, not an ongoing, maintained threat-tracking practice |

---

## 4. RESPOND

| # | Does the WWS... | Status | Notes |
|---|---|---|---|
| 4.A* | Have a written incident reporting procedure? | ✅ | ERP Section 3 (Escalation Chain) and Section 6 (Communication Plan) specify who to notify, in what order, and under what conditions — including CISA and EPA regulatory reporting paths |

---

## 5. RECOVER

| # | Does the WWS... | Status | Notes |
|---|---|---|---|
| 5.A | Have the ability to safely and effectively recover from an incident? | ⚠️ | ERP Section 5 (Recovery Procedures) defines a specific, dependency-ordered recovery sequence (HMI integrity first, then PLC, then Historian, then re-verification against the Artifact 3 baseline) — but like the IR plan itself, this has not yet been exercised |

---

## Summary

| Category | ✅ | ⚠️ | ❌ | N/A |
|---|---|---|---|---|
| Identify (7 items) | 0 | 2 | 5 | 1 |
| Protect (24 items) | 6 | 8 | 5 | 6 |
| Detect (1 item) | 0 | 1 | 0 | 0 |
| Respond (1 item) | 1 | 0 | 0 | 0 |
| Recover (1 item) | 0 | 1 | 0 | 0 |

*(Updated Aug 27, 2026 — item 2.A moved from ❌ to ✅ following credential remediation.)*

**Reading this honestly:** the ✅ items cluster almost entirely around network segmentation and documentation (2.F, 2.O, 2.P, 2.W, 2.X) — the exact area Artifact 3 targeted. That concentration is expected and should be stated plainly rather than implied as broader coverage than it is.

**The ❌ item carrying the most remaining weight** is now 2.H (MFA — absent entirely). 2.A (default passwords — Finding 1) was remediated Aug 27, 2026, closing what had been the most directly exploitable item on this checklist — no technical barrier had stood between an attacker with network access and full PLC control prior to the fix. Both 2.A and 2.H are Priority items on EPA's own Top Cyber Actions list; 2.A is now closed, and 2.H remains captured in the RRA's remediation roadmap.

**Governance and training items (1.A–1.I, 2.I–2.J) are the weakest category overall**, which is consistent with — and explains — the AWWA assessment's low Governance & Asset Management score (2) and the fact that this checklist result should not be read as a completed compliance exercise, but as the current gap analysis EPA's guidance intends it to produce: a prioritized list of what to fix next, not a passed/failed grade.

This checklist does not by itself satisfy the AWIA §2013 / SDWA §1433 requirement, which is addressed by the Risk & Resilience Assessment and Emergency Response Plan directly; EPA's guidance identifies this checklist as one voluntary tool that can inform those required documents, which is how it is used here.
