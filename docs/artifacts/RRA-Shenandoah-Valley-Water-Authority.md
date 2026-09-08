# Risk & Resilience Assessment
## Shenandoah Valley Water Authority — OT/ICS Environment

**Assessment basis:** AWIA §2013 Risk and Resilience Assessment requirements, informed by NIST 800-82 (ICS security), MITRE ATT&CK for ICS, and findings from Artifact 3 (Segmentation Before/After Assessment)
**Assessment date:** August 2026
**Prepared for:** Risk Committee / Board Review
**Source evidence:** Artifact 3 (Findings 1–4), Phase 3 network scan data, PLC egress anomaly investigation (Artifact 3, Appendix A)

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## 1. Asset Inventory

| Asset | Zone | IP | Function | Key Services | Auth Posture |
|---|---|---|---|---|---|
| PLC | Field/Control | 192.168.10.100 | Programmable logic controller — field process control | OpenPLC web (8080), Modbus TCP (502), EtherNet/IP (44818), S7comm (102), SSH (22) | **Default vendor credentials unchanged** (openplc/openplc) |
| HMI | Supervisory | 192.168.20.100 | Operator interface, supervisory control | Apache HTTP (80, unencrypted), SSH (22) | Not independently assessed in this cycle |
| Historian | Supervisory | 192.168.20.101 | Process data logging and retention | PostgreSQL (5432) | Database-native password auth; network-layer control added in remediation |
| Monitor | Monitoring | 192.168.40.100 | Network/security monitoring collection point | Syslog (514), SSH (22) | Not independently assessed |
| Engineering Workstation | Engineering | 192.168.30.100 | Engineering access, configuration | SSH (22) | Not independently assessed |
| Assessment Platform (Kali) | Engineering | 192.168.30.50 | Security assessment tooling | N/A — assessment host, not production | N/A |

All hosts run Ubuntu Linux with OpenSSH 9.6p1 (uniform patch baseline observed at assessment time). Asset criticality ranking: **PLC > Historian > HMI > Monitor/Engineering** — PLC compromise has the most direct path to a physical-process consequence.

---

## 2. Threat Analysis — MITRE ATT&CK for ICS Mapping

Threat paths below reflect the actual conditions found at Shenandoah Valley Water Authority prior to remediation, not a generic threat catalog.

| # | Attack Path | ATT&CK for ICS Technique(s) | Consequence if Successful |
|---|---|---|---|
| 1 | Attacker on Engineering zone port-scans Field/Control zone, enumerates PLC services | T0846 / T0846.001 — Remote System Discovery, Port Scan | Identifies PLC as a target; informs technique selection for #2–#4 |
| 2 | Attacker logs into OpenPLC web console using unchanged default credentials | T1694.001 — Insecure Credentials: Default Credentials; T0859 — Valid Accounts | Full administrative access to PLC configuration and logic without needing to defeat any technical control |
| 3 | Attacker sends unauthorized Modbus/EtherNet-IP commands directly to PLC | T0836 — Modify Parameter; T0855-class unauthorized command injection over exposed fieldbus protocol | Direct manipulation of control parameters — e.g., chemical dosing setpoints, pump/valve state — with physical-process consequence |
| 4 | Attacker exploits a known or unknown vulnerability in an exposed remote service (e.g., PostgreSQL, Werkzeug dev server) | T0866 — Exploitation of Remote Services | Code execution or data compromise on historian or PLC host |
| 5 | Attacker intercepts HMI session traffic (HTTP, unencrypted) on the Supervisory segment | T0842 — Network Sniffing | Credential or session exposure if HMI authentication is added without TLS |
| 6 | Attacker reaches historian database directly from Engineering, attempts credential brute-force or exploitation | T0866 — Exploitation of Remote Services; T0859 — Valid Accounts | Access to or corruption of historical process data; potential pivot point |
| 7 | Insider or malware-compromised engineering workstation uses legitimate remote access pathways to reach OT devices | T0886 — Remote Services | Lateral movement from a lower-trust zone into control-critical zones |

**Path criticality:** Paths #1→#2→#3 form a single continuous chain that was **fully viable before remediation** — an attacker with Engineering-zone access needed no exploit, only a port scan and a documented default password, to reach PLC control parameters. This chain is the single most important finding in this assessment: the barrier to a physical-process consequence was not a technical control at all, but the absence of one.

---

## 3. Vulnerability Mapping

| Finding (ref. Artifact 3) | Description | Severity | CVSS-style Rationale |
|---|---|---|---|
| Finding 1 | OpenPLC default credentials (openplc/openplc) live on network-reachable web interface | **High → Remediated (Aug 27, 2026)** | Was network-exploitable (AV:N), no privileges required (PR:N), no user interaction (UI:N), full admin scope — consistent with CVSS Base ≈ 9.x range for unauthenticated admin access. Default account replaced and verified dead; new credentials confirmed working via the same HTTP-request method used to identify the finding. |
| Finding 2 | Historian PostgreSQL access controlled at application layer only, not network layer | **High → Remediated (Aug 26, 2026)** | Network-exploitable path to a service with a known-broad vulnerability history; network segmentation is the standard compensating control and was absent. Segmentation added; network-layer path re-verified closed Aug 31, 2026 |
| Finding 3 | Firewall configuration existed but was not enforced (PLC, Historian) | **Medium (methodological) → Remediated (Aug 26, 2026)** | Not directly exploitable itself, but it is the root enabler of Findings 1 and 2's network reachability — addressed via segmentation remediation. Live enforcement confirmed and re-verified Aug 31, 2026 |
| Finding 4 | HMI web interface unencrypted (HTTP only) | **Medium** | Passive network position required (higher attacker cost than #1/#2); confidentiality impact on any future HMI credentials |
| Finding 3.1 | PLC-to-historian egress anomaly — root cause identified (Artifact 3, Appendix A) | **Low / Informational** | Not an exploitable vulnerability — it is a control that fails *closed*, not open. Root cause diagnosed (Proxmox `firewall=1` + `br_netfilter` NAT interaction); symptom cleared by a host reboot but the underlying condition persists and could recur. Flagged for engineering follow-up (permanent remediation), not risk-elevating on its own |

---

## 4. Risk Matrix (Likelihood × Impact)

Likelihood and impact scored 1 (low) – 5 (high). Likelihood reflects **pre-remediation** conditions unless noted; Impact reflects physical/operational consequence to Shenandoah Valley Water Authority.

| Risk | Likelihood | Impact | Score | Rating |
|---|---|---|---|---|
| Unauthorized PLC parameter modification via default creds + open Modbus/EtherNet-IP | ~~5~~ **2** | 5 | ~~25~~ **10** | ~~Critical~~ **Medium** *(re-scored Aug 27, 2026 — default credential path closed; see Section 6)* |
| Historian compromise via unrestricted network path to PostgreSQL | 4 | 3 | 12 | **High** |
| Lateral movement from Engineering into Field/Control zone generally | 4 | 4 | 16 | **High** |
| HMI session interception (unencrypted HTTP) | 2 | 3 | 6 | **Medium** |
| Configuration-vs-enforcement gap recurring on a future host | 3 | 3 | 9 | **Medium** |
| PLC-historian egress anomaly (Finding 3.1) exploited as an attack vector | 1 | 2 | 2 | **Low** |

**Top risk driver:** the Critical-rated risk (PLC parameter modification) scores highest not because the technique is sophisticated — it is not — but because likelihood and impact are simultaneously maximal: the path required no special access and the consequence reaches the physical process directly. This is the risk profile most characteristic of OT environments generally, and is why segmentation was prioritized as the first remediation action ahead of the credential fix itself (a network path with a bad password is worse than a bad password with no path).

---

## 5. Compliance Mapping

| Framework | Relevant Provision | Shenandoah Valley Status |
|---|---|---|
| AWIA §2013 | Risk and Resilience Assessment — malevolent acts and natural hazards to system | This document, plus Artifact 3, constitutes the cybersecurity-malevolent-act component of the RRA requirement |
| EPA Cybersecurity Checklist | Network segmentation between IT/OT and within OT zones | **Substantially improved** post-remediation (Artifact 3, Section 7) — Field/Control and Historian access from Engineering now default-deny |
| EPA Cybersecurity Checklist | Default credential elimination | **✅ Remediated (Aug 27, 2026)** — default account replaced and verified; see Section 6 and EPA Checklist item 2.A |
| NIST 800-82 | Network architecture — Purdue Model zone separation | **Aligned** — remediation followed Purdue Level 1/2/3 separation explicitly (Artifact 3, Section 9) |
| NIST 800-82 | Least-privilege conduit design | **Aligned** — only two conduits authorized post-remediation (HMI↔PLC, HMI↔Historian), both individually verified |

---

## 6. Residual Risk After Segmentation

**Reduced but not eliminated:**
- PLC and historian are no longer reachable from Engineering. The Critical-rated risk in Section 4 is reduced in *likelihood* (attacker must now first compromise the HMI or physically access the Supervisory zone) but not in *impact* — if the HMI itself is compromised, the same PLC parameter-modification path remains available, since HMI→PLC is an authorized conduit by design.
- **New consideration introduced by segmentation:** the HMI is now a more concentrated point of trust than before, since it is the sole authorized path to both the PLC and the historian. This does not argue against the remediation — concentrating trust into one well-defended chokepoint is preferable to leaving it distributed across an open network — but it does mean the HMI itself should be prioritized for the credential and hardening review that Finding 1 flagged for the PLC.
- **Update, Aug 27, 2026:** Finding 1 (default OpenPLC credentials) has been remediated — the default account was replaced and verified dead via the same HTTP-request method used to originally identify it. This closes what had been the top-scored risk in Section 4. Residual exposure at the HMI still exists in principle (an attacker reaching the HMI now needs to defeat a real credential rather than a documented default, which is a materially higher bar, not a zero-risk state) — the HMI's own credential posture has not yet been independently reviewed and remains tracked as RRA Priority 2 below.
- Finding 4 (unencrypted HMI web interface) is unaffected by segmentation and remains open.

---

## 7. Prioritized Remediation Roadmap

| Priority | Action | Addresses | Effort |
|---|---|---|---|
| **1 — Immediate** | ~~Change OpenPLC default credentials~~ **✅ Complete (Aug 27, 2026)** — default account replaced, verified via login test | Finding 1 | Low |
| **2 — Immediate** | Extend credential review to HMI, given its new role as the sole conduit to PLC and historian | Residual risk (Section 6) | Low |
| **3 — Near-term** | Add TLS to HMI web interface | Finding 4 | Medium |
| **4 — Near-term** | Formalize a live-enforcement verification step (not just config review) into any future firewall change process, per Finding 3's lesson | Finding 3 | Low (procedural) |
| **5 — Medium-term** | Implement a permanent fix for the PLC-to-historian egress anomaly (Finding 3.1) so the underlying condition cannot recur | Appendix A anomaly | Medium (root cause identified; requires a platform-level fix — e.g., disabling bridge-netfilter or restructuring the NAT path — not further investigation) |
| **6 — Ongoing** | Re-run the Artifact 3 scan methodology periodically (quarterly suggested) to catch configuration drift, especially the configuration-vs-enforcement failure mode identified in Finding 3 | All findings | Low (repeatable, evidence methodology already built) |

Priorities 1–2 are quick wins that close the highest-severity residual risk with minimal effort and should not wait for the broader Phase 4 package to complete. Priorities 3–4 are near-term hardening. Priority 5 is explicitly not urgent — Artifact 3 documents why the current HMI-relay architecture is an acceptable, arguably tighter alternative to the original direct-conduit design.

---

## Appendix: Cross-Reference to Artifact 3

This RRA is derived from and should be read alongside Artifact 3 (Segmentation Before/After Assessment), which contains the underlying scan evidence, the full elimination methodology for the PLC egress anomaly, and the before/after port-state data referenced throughout Sections 2–4 above. This document does not duplicate that evidence; it applies formal risk scoring and compliance mapping to it.
