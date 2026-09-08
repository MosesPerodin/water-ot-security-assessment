# OT Cybersecurity Assessment
## Shenandoah Valley Water Authority
### Final Report

**Prepared for:** General Manager, Board of Directors, and Plant Operations Leadership
**Assessment period:** August 2026
**Scope:** Operational technology network serving water treatment process control
**Classification:** Internal — contains security-sensitive information

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## 1. Purpose of This Report

This report consolidates the findings of a technical security assessment of Shenandoah Valley Water Authority's operational technology (OT) network — the systems that monitor and control the physical treatment process.

It is written for decision-makers rather than engineers. Where technical detail is necessary, it is explained in terms of operational consequence: what could happen to the treatment process, to water quality, to service continuity, and to the Authority's regulatory standing. Supporting technical documentation is indexed in Section 8 for staff who need it.

Three things this report is not:

- **It is not a compliance certification.** No such certification exists for water-sector cybersecurity. This assessment maps to recognized frameworks (EPA, AWWA, NIST, IEC) to structure findings, not to issue a passing grade.
- **It is not a claim that the Authority is now secure.** It documents specific improvements that were made and verified, and specific gaps that remain open.
- **It is not exhaustive.** Section 7 states plainly what was not assessed and what could not be resolved.

---

## 2. Executive Summary

**The assessment found that the Authority's process control network was, at the outset, effectively flat.** An engineering workstation — the kind of machine a contractor or a compromised employee account might reach — had direct network access to the programmable logic controller (PLC) that operates treatment equipment, including access to its unprotected administrative interface. That interface was still using the vendor's published default password.

In plain terms: at the start of this engagement, anyone who gained access to a single engineering workstation could have reached the controller governing physical treatment operations and logged into it using a password printed in the product's public documentation. No exploit, no specialized tooling, and no advanced skill would have been required.

**Both of those conditions have been corrected and independently verified.**

| | Before | After |
|---|---|---|
| PLC network exposure from Engineering zone | 5 open service ports, including industrial control protocols | 1 (remote administration only) |
| Historian database exposure from Engineering zone | Directly reachable | Closed |
| PLC administrative credentials | Vendor default, confirmed working | Replaced, old credentials confirmed dead |
| Network segmentation | None enforced | Four zones, default-deny, verified by testing |

**What this reduces:** the primary risk in a water treatment environment is not data theft — it is unauthorized change to the physical process. Chemical dosing setpoints, pump states, and valve positions are the assets that matter, because those are the ones connected to public health. The remediation above removes the most direct path an intruder had to those controls.

**What remains open, stated plainly:**

1. **Multi-factor authentication is not implemented anywhere in the OT environment.** This is the highest-priority remaining gap and appears on the joint EPA/CISA/FBI list of recommended priority actions for water systems.
2. **The operator interface (HMI) has not received the same credential review the PLC has.** Because segmentation now routes all control traffic through the HMI, it has become a more concentrated point of trust than it was before. That is a deliberate and defensible design, but it means the HMI's own security posture now carries more weight.
3. **No tested backup and recovery capability exists** for control system configuration or process data.
4. **Governance is the weakest area overall.** There is no named individual accountable for OT cybersecurity, no formal change control for OT systems, and no scheduled reassessment. The technical improvements documented here are a one-time project, not yet a maintained practice.

**The single most important recommendation in this report is #4 below** — not because it is the most technical, but because without it, the improvements described here will erode. Firewall rules drift, staff turn over, new equipment arrives with new default passwords. The gap that produced the original findings was not a lack of technical skill; it was the absence of a process that would have caught them.

---

## 3. How the Assessment Was Conducted

**Approach:** the assessment established a documented baseline of the network's actual state, implemented segmentation controls, and then repeated identical tests to verify the controls worked as intended. Findings are based on what was measured, not on configuration review alone.

**That distinction proved important.** In three separate instances during this engagement, security configuration existed in the correct files, appeared correct on inspection, and was not actually in effect:

- Firewall rules for two critical hosts were present in configuration but never enforced by the platform
- The intrusion detection system reported itself as healthy for days while performing no packet inspection at all
- A custom detection rule was silently ignored due to a file path mismatch, with no error reported

**None of these would have been caught by reviewing documentation or asking staff whether controls were in place.** All three were caught by testing whether the control actually did what it claimed. This is the single most transferable methodological point in this report, and it directly informs Recommendation 4.

**Testing restraint:** all network scanning used deliberately conservative timing. Aggressive scanning is known to destabilize legacy industrial controllers, and an assessment that disrupts the process it is meant to protect has failed regardless of what it finds. This constraint is documented rather than assumed.

---

## 4. Consolidated Findings

Findings are ordered by operational consequence, not by technical severity score.

### Finding A — Default vendor credentials on the process controller *(Remediated)*

The PLC's web administrative interface accepted the manufacturer's published default username and password. This was confirmed by direct test, not inferred from configuration.

**Consequence had it been exploited:** full administrative access to the device that governs treatment equipment. An intruder could have altered control logic or setpoints. The Oldsmar, Florida incident of 2021 — in which a caustic soda setpoint at a water plant was reportedly increased roughly one hundred-fold before an operator reversed it — is the reference case for what this class of access enables. (That incident's attribution was later disputed by the FBI; the technical conditions that made it possible were not.)

**Status:** remediated August 27, 2026. Default account replaced; old credentials verified non-functional; new credentials verified working.

### Finding B — Flat network permitting direct Engineering-to-controller access *(Remediated)*

Industrial control protocols (Modbus, EtherNet/IP, S7comm) and the PLC's administrative interface were all directly reachable from the Engineering zone.

**Consequence had it been exploited:** these protocols were designed for trusted networks and generally include no authentication. Reachability is effectively equivalent to control. Compromise of any engineering workstation would have provided a direct path to process manipulation.

**Status:** remediated. Verified by repeat testing: four of five previously-open ports now filtered, remote administration only.

### Finding C — Historian database directly exposed *(Remediated)*

The process historian's database port was reachable from the Engineering zone, protected only by application-level authentication.

**Consequence:** the historian holds the operational record — the evidence trail for water quality compliance and the data operators rely on to detect slow-developing equipment problems. Quiet corruption of this record is arguably more damaging long-term than an obvious outage, because it may go undetected until it matters.

**Status:** remediated. Database port closed at the network layer.

### Finding D — No encryption on the operator interface *(Open)*

The HMI web interface transmits over unencrypted HTTP. Credentials and process data traverse the network in plain text.

**Consequence:** anyone able to observe network traffic on the supervisory segment can capture operator credentials. Given that the HMI is now the sole authorized path to both the PLC and historian, those credentials are more valuable than they were before segmentation.

**Recommended action:** enable TLS on the HMI interface.

### Finding E — No multi-factor authentication *(Open)*

No OT system requires a second authentication factor. Remote administrative access uses cryptographic keys, which is stronger than passwords alone but does not constitute MFA.

**Consequence:** every remaining access control in the environment depends on a single secret. Finding A demonstrated what happens when a single secret is weak; MFA is the control that limits the damage when one is compromised rather than assuming none ever will be.

**Recommended action:** implement MFA for all remote and administrative OT access. This is a designated priority action in joint EPA/CISA/FBI guidance for water systems.

### Finding F — Intrusion detection has significant coverage gaps *(Open)*

An intrusion detection system was deployed and verified functional. Two coverage limitations were identified through testing:

- **It cannot see traffic that the firewall blocks.** A five-port scan against the monitored host produced records for only the one permitted port. Reconnaissance activity — an attacker probing for a way in — is invisible to the sensor in its current position.
- **The standard commercial ruleset does not detect password-guessing attacks.** Eight consecutive failed login attempts generated zero alerts. Verification against the loaded ruleset confirmed no rate-based detection rule exists in the default configuration.

**Consequence:** the Authority currently has no automated alerting for either reconnaissance against OT assets or credential attacks against them. Given that credential weakness was the highest-severity finding in this assessment, the absence of detection for exactly that attack class is significant.

**Recommended action:** configure threshold-based authentication-failure detection, and establish sensor visibility ahead of firewall filtering.

### Finding G — No tested backup or recovery capability *(Open)*

No backup schedule exists for control system configuration, control logic, or historian data, and no recovery has been tested.

**Consequence:** the Emergency Response Plan developed during this engagement defines a recovery sequence. That plan assumes something to recover from. Without tested backups, recovery from a destructive incident or significant equipment failure would be reconstruction from memory.

### Finding H — Governance and accountability gaps *(Open)*

No individual holds named responsibility for OT cybersecurity. No change-control process governs new services or credentials on OT systems. No reassessment cadence exists.

**Consequence:** this is the condition that allowed Findings A through C to persist undetected. Technical remediation without governance produces a secure snapshot, not a secure system.

---

## 5. Security Maturity Assessment

Scored against AWWA's cybersecurity maturity framework. Scale: 1 (Ad hoc) to 5 (Continuous improvement).

| Area | Before | Current | 12-Month Target |
|---|---|---|---|
| Network Security & Segmentation | 1 | **3** | 4 |
| Monitoring & Incident Response | 2 | **3** | 4 |
| Access Control & Authentication | 1 | **2** | 4 |
| Governance & Asset Management | 2 | **2** | 3 |
| Supply Chain & Third-Party Risk | 1 | **1** | 2 |

**Reading this table honestly:** the improvement is real and concentrated in one area. Network segmentation moved from Ad hoc to Defined, with evidence rather than self-report.

**It is deliberately not scored higher than 3.** A score of 4 implies a managed, continuously-verified practice. What exists today is a well-executed one-time project with a recommended quarterly re-verification that has not yet run a second time. Scoring it higher would overstate the Authority's actual position, and any subsequent reviewer would find that out.

The areas that remain at 1 and 2 — governance, access control, supply chain — are the ones Recommendations 1 through 4 address.

---

## 6. Prioritized Recommendations

| # | Recommendation | Addresses | Effort | Priority |
|---|---|---|---|---|
| 1 | Implement multi-factor authentication for OT administrative access | Finding E | Medium | **Immediate** |
| 2 | Conduct credential review of the HMI, matching the PLC remediation | Findings A, D | Low | **Immediate** |
| 3 | Enable TLS encryption on the operator interface | Finding D | Low | Near-term |
| 4 | Assign named OT cybersecurity accountability and establish change control | Finding H | Low | **Immediate** |
| 5 | Establish and test a backup and recovery capability | Finding G | Medium | Near-term |
| 6 | Configure authentication-failure detection alerting | Finding F | Low | Near-term |
| 7 | Institute quarterly re-verification against the documented baseline | Findings B, C, H | Low | Ongoing |
| 8 | Exercise the Emergency Response Plan via tabletop drill | — | Low | Near-term |
| 9 | Establish vendor vulnerability tracking for OT software components | — | Medium | Medium-term |

**On Recommendation 4:** it is listed as low effort and immediate priority because it is neither expensive nor technical, and because every other recommendation depends on someone owning it. Assigning accountability costs a decision, not a budget.

**On Recommendation 7:** the assessment produced a documented baseline of exactly what the network should look like. Re-running the same tests quarterly and comparing against that baseline is the difference between a security posture that holds and one that quietly degrades. This is what moves segmentation from a score of 3 to a 4.

---

## 7. Limitations and Open Items

Stated explicitly so that this report is not read as covering more than it does.

**Not assessed:**
- **Physical security** — the assessment was conducted against a virtualized representation of the control network and did not evaluate physical access to field equipment, control rooms, or server hardware. A separate facility assessment would be required.
- **Field instrumentation** — sensors, actuators, and drives downstream of the PLC were not independently examined.
- **Business/IT network** — scope was limited to OT. The IT/OT boundary warrants separate review.
- **Personnel and training** — security awareness and OT-specific training were out of scope.

**Unresolved technical items:**
- **One network path anomaly** prevented the controller from being configured with direct communication to the historian under the new firewall policy. The architecture routes around this via the operator interface, which is a defensible design, but the underlying cause was not fully determined. Systematic elimination of fourteen hypotheses is documented.
- **A related platform-level issue** was subsequently identified and confirmed by controlled testing, which explains a family of connectivity failures observed during the engagement. This is a virtualization platform behavior rather than a misconfiguration by Authority staff.
- **Traffic mirroring for intrusion detection was not completed.** Four distinct faults were identified and three resolved; the fourth remains open. Work was suspended in favor of documenting findings rather than extending the engagement. The specific next diagnostic step is documented for future resumption.

**These items are disclosed rather than omitted.** An assessment that reports only what it resolved provides no basis for judging what it missed.

---

## 8. Supporting Documentation

| Document | Contents |
|---|---|
| Segmentation Assessment | Full technical detail, before/after evidence, network architecture, unresolved anomaly appendix |
| Asset Inventory & Criticality Analysis | Complete asset register with consequence-based criticality ranking and methodology |
| Risk & Resilience Assessment | Threat analysis, risk matrix, residual risk, remediation roadmap (supports AWIA §2013) |
| Emergency Response Plan | Incident classification, detection triggers, escalation, containment, recovery, communications |
| AWWA Self-Assessment | Maturity scoring with justification per section |
| EPA Cybersecurity Checklist | Item-by-item status against EPA 817-B-23-001 |
| NIST 800-53 / IEC 62443 Crosswalk | Control-level mapping with evidence citations |
| Threat Detection Assessment | IDS deployment, coverage findings, verification methodology |
| Threat Case Study | MITRE ATT&CK analysis of a comparable water-sector incident |

---

## 9. Closing Assessment

Shenandoah Valley Water Authority's process control network is materially more defensible than it was at the start of this engagement. The two most direct paths to unauthorized process control — a flat network and a default password — have been closed, and the closure has been verified by testing rather than asserted.

The remaining work is less dramatic and more durable. Multi-factor authentication, tested backups, named accountability, and a re-verification cadence are unglamorous controls. They are also the ones that determine whether the improvements in this report are still true in a year.

The Authority's position today is best summarized as: **the most urgent technical gaps are closed and proven closed; the practices that would keep them closed do not yet exist.** Recommendations 1, 2, and 4 address that directly and require modest effort.

---

*This report reflects the state of the assessed environment as of August 30, 2026. Security posture is time-dependent; findings should be re-verified on the cadence recommended in Section 6.*
