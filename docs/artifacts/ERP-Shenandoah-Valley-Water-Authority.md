# Emergency Response Plan — OT/ICS Environment
## Shenandoah Valley Water Authority

**Scope:** Field/Control (PLC), Supervisory (HMI, Historian), Engineering, and Monitoring zones
**Format:** Incident response playbook — checklist/procedure, for use during an active event
**Basis:** Segmented architecture per Artifact 3; conduit map: HMI↔PLC (Modbus/502), HMI↔Historian (PostgreSQL/5432)

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## 1. Incident Classification

Classify by which security property is affected and the OT consequence it maps to. Use the **highest** applicable classification if an incident spans categories.

| Class | Definition | OT Consequence | Example Trigger |
|---|---|---|---|
| **Integrity** | Unauthorized modification of control logic, setpoints, or process data | Incorrect chemical dosing, valve/pump misoperation, falsified historian records | Unexpected Modbus write to PLC; historian values inconsistent with known process state |
| **Availability** | Loss of access to control, monitoring, or supervisory function | Loss of visibility into or control over treatment process; inability to respond to a developing condition | HMI unreachable; PLC stops responding; monitoring feed goes dark |
| **Confidentiality** | Unauthorized disclosure of credentials, network topology, or process data | Enables a future, more damaging integrity/availability event; regulatory exposure if customer or compliance data involved | Unexpected outbound connection from OT zone; credential found exposed |

**Severity tiers within each class:**
- **Sev 1 (Critical):** Confirmed or suspected unauthorized access to PLC; any indication of control-logic or setpoint tampering
- **Sev 2 (High):** Confirmed unauthorized access to Historian or HMI; unexplained conduit traffic outside the two authorized paths
- **Sev 3 (Moderate):** Anomalous but unconfirmed activity (e.g., unexpected port activity, failed connection attempts from an unauthorized source)

---

## 2. Detection Triggers

| Trigger | Source | Action |
|---|---|---|
| Nmap baseline deviation — a port opens on PLC/Historian that Artifact 3's after-state shows should be filtered | Periodic re-scan (Priority 6, RRA remediation roadmap) | Treat as Sev 1 if on PLC, Sev 2 if on Historian |
| Connectivity-probe failure on an authorized conduit (HMI→PLC or HMI→Historian) | Scheduled conduit health check | Treat as Sev 2 (Availability) — do not assume malicious until triaged |
| Service fingerprint change on any OT host (e.g., SSH banner, OpenPLC version string changes unexpectedly) | Scan comparison against Artifact 3 baseline | Treat as Sev 2 pending investigation |
| Login to OpenPLC dashboard from an unexpected source or at an unexpected time | Manual log review (until automated monitoring is in place — see RRA Priority 6) | Treat as Sev 1 |
| Traffic observed on a conduit or path not in the authorized list (Section in Artifact 3, "Segmentation Design") | Monitoring zone (192.168.40.100) | Treat as Sev 1 if touching Field/Control, Sev 2 otherwise |

---

## 3. Escalation Chain

| Step | Who | Trigger |
|---|---|---|
| 1 | On-call assessor/engineer (whoever holds current OT security responsibility) | Any Sev 1–3 detection |
| 2 | Plant operations lead | Sev 1 immediately; Sev 2 within 1 hour; Sev 3 at next check-in |
| 3 | Utility CISO / IT security lead | Sev 1 immediately; Sev 2 if unresolved within 4 hours |
| 4 | External: CISA (via 888-282-0870 or [cisa.gov/report](https://cisa.gov/report)) | Sev 1 confirmed, or any suspected nation-state/criminal activity |
| 4 | External: EPA regional office | Any incident with actual or potential impact to water quality or service continuity, per AWIA reporting expectations |
| 5 | Public communication (see Section 6) | Only after Steps 1–4 have assessed actual operational/public-health impact |

**Do not skip Step 1.** Even a confirmed Sev 1 goes through the on-call assessor first — this preserves evidence-handling discipline (chain of custody, matching Artifact 3's methodology) before the incident becomes a wider-visibility event.

---

## 4. Containment Procedures

Ordered from least to most disruptive. Use the lowest-numbered step that stops the incident — do not jump to zone isolation if a narrower rule change suffices.

1. **Revoke the specific credential or session** if the incident is a compromised account (e.g., OpenPLC default credentials, per Finding 1) — fastest, least disruptive
2. **Lock down the specific conduit rule** in the affected `.fw` file (e.g., temporarily remove HMI's permitted-source rule in `102.fw` if Historian is the target) — isolates without killing the whole zone
3. **Set the affected VM's firewall to full default-deny** (`policy_in: DROP`, no exceptions) if the specific-rule lockdown in Step 2 isn't sufficient or the threat is unclear
4. **Isolate the entire zone** at the Proxmox bridge level (disable the bridge or set every host's NIC to `firewall=0` with a blanket deny — inverse of the anomaly investigated in Artifact 3, Appendix A) — last resort, used only for a confirmed Sev 1 with active, ongoing control-logic tampering
5. **Physical isolation** (disconnect network cable / power down affected VM) if steps 1–4 are insufficient or the platform-level firewall itself is suspected compromised

**Before any containment step:** capture current state (running `nmap` scan, `iptables -L`, active connections) using the same methodology as Artifact 3 — this becomes incident evidence, not just operational cleanup.

---

## 5. Recovery Procedures

Recovery order follows the conduit dependency chain established in Artifact 3 — Historian and PLC both depend on HMI being trustworthy before either resumes.

1. **Verify HMI integrity first.** Since HMI is the sole conduit to both PLC and Historian post-segmentation, it must be confirmed clean before anything reconnects through it.
2. **Restore PLC connectivity** (if isolated) — re-enable the HMI→PLC Modbus conduit only, matching the exact rule from `101.fw`/`102.fw` used in Artifact 3's after-state. Do not restore broader connectivity as a shortcut.
3. **Restore Historian connectivity** — same principle, HMI→Historian PostgreSQL conduit only.
4. **Re-run the Artifact 3 scan methodology** against the recovered environment and diff against the documented after-state (Section 6 of Artifact 3). Any deviation is unresolved and blocks declaring recovery complete.
5. **Change any credential involved in the incident** before returning the asset to production, even if the credential wasn't confirmed compromised — precautionary, matches the credential-hygiene gap already flagged in Finding 1.
6. **Historian data validation** — if the incident involved potential data integrity loss, validate recent historian records against known process behavior before treating them as reliable for compliance or operational reporting.

---

## 6. Communication Plan

| Audience | When | Content |
|---|---|---|
| **Internal — operations staff** | Immediately upon Sev 1/2 declaration | What's affected, what to avoid doing (e.g., don't attempt manual PLC access during containment), expected timeline |
| **Internal — leadership** | Per escalation chain (Section 3) | Severity, consequence assessment, containment status |
| **Regulatory — EPA** | Per AWIA reporting expectations, once impact (actual or potential) to water quality/service is assessed | Factual incident summary; avoid speculation on cause before investigation concludes |
| **Regulatory — CISA** | Sev 1 confirmed or suspected external actor | Per CISA's standard incident reporting process |
| **Public** | Only if there is an actual or credibly imminent impact to water service or quality, and only after internal/regulatory steps above | Coordinated through leadership and, where applicable, EPA guidance — no unilateral public statements from operational or security staff |

**General rule throughout:** communicate what is confirmed, flag what is still under investigation as such, and do not commit to a root cause or a "this is now resolved" statement until the Section 5 recovery validation is complete. This mirrors the assessment discipline used throughout Artifact 3 — findings are stated precisely as their evidence supports, whether that means confirming a root cause (as with the PLC-to-historian egress anomaly, Finding 3.1) or leaving something genuinely open, rather than smoothing over either direction.
