# AWWA Cybersecurity Self-Assessment
## Shenandoah Valley Water Authority

**Framework:** American Water Works Association security guidance, maturity-level scoring
**Scale:** 1 (Ad hoc) – 2 (Repeatable) – 3 (Defined) – 4 (Managed) – 5 (Continuous improvement)
**Evidence basis:** Artifact 3 (Segmentation Assessment), RRA, ERP
**Scored:** post-remediation (current) state, with pre-remediation shown for contrast where the delta matters

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## Scoring Summary

| Section | Pre-Remediation | Current | Target (12-mo) |
|---|---|---|---|
| 1. Governance & Asset Management | 2 | 2 | 3 |
| 2. Network Security & Segmentation | 1 | **3** | 4 |
| 3. Access Control & Authentication | 1 | 2 | 4 |
| 4. Monitoring & Incident Response | 2 | 3 | 4 |
| 5. Supply Chain & Third-Party Risk | 1 | 1 | 2 |
| 6. Physical Security | N/A | N/A | N/A |

**Reading this table:** the largest single-section jump — Network Security & Segmentation, 1→3 — is the direct result of Artifact 3's remediation work. It is also, deliberately, not scored higher than 3: a one-time hardening project, however well-executed and well-evidenced, is not yet a *managed* or *continuously improving* practice. That distinction matters and is explained in Section 2 below.

---

## 1. Governance & Asset Management — **Score: 2 (Repeatable)**

**What supports this score:**
- A documented, accurate asset inventory exists (RRA Section 1) — host, zone, IP, function, and key services are known and current
- Asset criticality has been informally ranked (PLC > Historian > HMI > Monitor/Engineering) based on consequence, not just convenience

**What holds this at Repeatable rather than Defined:**
- Asset inventory currently exists as a point-in-time artifact from this assessment cycle, not a continuously maintained system of record
- No formal asset-ownership or change-management process governs when new services get added to OT hosts (this is precisely how Finding 3's configuration-vs-enforcement gap and Finding 1's unrotated default credential both went unnoticed for as long as they did)
- No documented governance policy requiring periodic reassessment exists yet — this document is itself the first instance, not part of an established cadence

**To reach Defined (3):** formalize the RRA's asset inventory into a maintained record, and establish an ownership/change-control process so any new service or credential on an OT host has an accountable owner and a review step before going live.

---

## 2. Network Security & Segmentation — **Score: 3 (Defined)**

**What supports this score:**
- Zone-based segmentation is implemented and independently verified (Artifact 3, Sections 5–7): PLC's exploitable surface from Engineering reduced 5 ports → 1; Historian's database port closed to Engineering entirely
- Segmentation design follows a recognized reference model (Purdue/IEC 62443 zones-and-conduits), not an ad hoc rule set
- Every permitted cross-zone conduit is explicit and individually tested, rather than inferred from configuration intent — this is the specific practice that closes the gap identified in Finding 3

**Why this is capped at 3, not 4:**
- The segmentation was validated as a **project**, not yet operated as a **standing practice**. There is no scheduled re-verification cadence yet (RRA Priority 6 recommends quarterly re-scans, but this has not yet run a second time)
- Finding 3 (configuration existing without enforcement) demonstrated that a firewall policy can silently fail without a live-verification step catching it — until periodic re-verification is actually operating, the same class of silent failure could recur undetected
- The unresolved PLC-to-historian egress anomaly (Artifact 3, Appendix A) means one part of the intended architecture was worked around rather than fully realized as designed, even though the workaround is defensible

**To reach Managed (4):** operationalize the quarterly re-scan (RRA Priority 6) as a scheduled, owned process with defined pass/fail criteria against the Artifact 3 after-state baseline, not a one-time comparison.

---

## 3. Access Control & Authentication — **Score: 2 (Repeatable)**

**What supports this score:**
- SSH access across OT hosts uses key-based authentication (established during Phase 2), not passwords
- Sudo access has been deliberately scoped and cleaned up after use (sudoers cleanup, verified via `sudo -n` failure test) rather than left standing indefinitely

**Update, Aug 27, 2026 — Finding 1 remediated:** OpenPLC's default vendor credentials (`openplc`/`openplc`) have been replaced. The default account was confirmed dead and the new credentials confirmed working, using the same HTTP-request method that originally identified the finding. RRA Priority 1 is now complete.

**What still holds this at Repeatable, not yet Defined:**
- The RRA's residual-risk analysis flags that HMI — now the sole conduit to both PLC and Historian post-segmentation — has **not yet had an equivalent credential review performed** (RRA Priority 2, currently unaddressed)
- No formal password/credential policy or rotation schedule exists across OT assets — the PLC fix was a targeted, single-host remediation, not yet a standing practice

**To reach Defined (3):** complete Priority 2 of the RRA remediation roadmap (HMI credential review) and establish a minimum credential standard applied consistently across all OT hosts, not host-by-host as issues surface.

---

## 4. Monitoring & Incident Response — **Score: 3 (Defined)**

**What supports this score:**
- A dedicated monitoring zone and host exist and were confirmed correctly segmented throughout the assessment (no regression observed, Artifact 3 Section 6.3)
- A written, structured Emergency Response Plan now exists, with explicit incident classification, detection triggers, escalation chain, containment procedures ordered by disruption level, recovery sequencing, and a communication plan
- The ERP's detection triggers are tied to concrete, repeatable evidence (Artifact 3's documented after-state baseline), not vague anomaly language

**What holds this at Defined, not Managed:**
- The monitoring host currently collects syslog but does not yet have active alerting or automated detection tied to the ERP's trigger conditions (e.g., "port opens that should be filtered" is currently a manual re-scan comparison, not an automated alert)
- The ERP has not yet been exercised via a tabletop or live drill — a written plan that has never been tested carries meaningfully more execution risk than one that has

**To reach Managed (4):** implement automated alerting against the Artifact 3 baseline (closing the manual-comparison gap) and run at least one tabletop exercise against the ERP's Sev 1 scenario.

---

## 5. Supply Chain & Third-Party Risk — **Score: 1 (Ad hoc)**

**What supports even this score:**
- Software components in use are identified and versioned (OpenPLC, PostgreSQL, Apache, OpenSSH — RRA Section 1), which is a prerequisite for any supply-chain risk work, even if not yet acted on

**Why this remains Ad hoc:**
- No vendor risk assessment has been performed on OpenPLC or any other third-party component in the environment
- No process exists for tracking upstream vulnerability disclosures against the specific versions in use (e.g., OpenSSH 9.6p1, PostgreSQL, Werkzeug 2.3.7)
- This category was explicitly out of scope for the Phase 3 technical assessment and has received no dedicated attention to date

**To reach Repeatable (2):** establish a basic practice of checking installed component versions against CVE feeds or vendor advisories on a defined cadence — this does not require a mature program, only a first repeatable step.

---

## 6. Physical Security — **Not Scored (Out of Scope)**

The lab environment on which this assessment is based is virtualized (Proxmox host, single physical location) and does not model physical access controls to field equipment, control rooms, or the historian/HMI hosting environment in a way that produces meaningful maturity evidence. This section is intentionally left unscored rather than assigned a placeholder value, consistent with the AWWA framework's allowance for scoping notes. A physical security assessment would need to be conducted against Shenandoah Valley Water Authority's actual facility, separate from this technical assessment.

---

## Overall Assessment

Shenandoah Valley Water Authority's OT security posture improved measurably and verifiably in the area directly targeted by this assessment cycle — network segmentation moved from Ad hoc (1) to Defined (3), with hard evidence rather than a self-reported claim. That improvement is real and should be represented as such.

It should equally be represented honestly that **this improvement is narrow and project-based**, not yet reflected in the surrounding practices — governance and supply-chain risk remain at or near baseline maturity. Finding 1 (default OpenPLC credentials), which had been the highest-severity open item, was remediated on August 27, 2026, closing the top-scored risk in the RRA. A board or executive audience should read this assessment as: *the two highest-risk technical gaps — open network paths and a default credential — have both been closed and independently verified closed, and the next-highest-priority items (HMI credential review, MFA, encryption in transit) are now clearly identified and sequenced* — not as a broad security-maturity uplift across the organization.
