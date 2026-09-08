# OT Asset Inventory & Criticality Analysis
## Shenandoah Valley Water Authority

**Assessment type:** Foundational asset inventory and consequence-based criticality ranking
**Scope:** OT network assets across Field/Control, Supervisory, Engineering, and Monitoring zones
**Prepared for:** Internal risk and asset management reference; input to Risk & Resilience Assessment
**Relationship to other artifacts:** This document is the foundational input the RRA's Section 1 summarizes — this is the full methodology and reasoning behind that summary table, not a duplicate of it

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## 1. Executive Summary

Before any risk can be assessed or any control prioritized, an assessor must know what exists and what happens if each thing fails. This document catalogs every OT asset at Shenandoah Valley Water Authority's assessed environment and ranks each by operational consequence — not by technical sophistication, network position, or ease of assessment, but by what breaks in the physical world if the asset is compromised, taken offline, or has its data corrupted.

**Why this matters operationally, not just technically:** a network engineer ranks assets by exposure — what's reachable, what's patched, what's loud on the wire. An operator ranks them by consequence — what happens to the water. Ten years running pretreatment and anaerobic digestion means the criticality ranking below isn't a guess dressed up as methodology; it reflects what actually causes an operational upset versus what's merely inconvenient. This is the specific value an operations background adds to an assessment that a pure IT-background assessor cannot replicate without borrowing it from someone like the person writing this.

**Result:** five assets identified and ranked. The PLC is unambiguously the highest-criticality asset — it is the only asset with a direct, unmediated path to a physical-process consequence. Everything else matters because of what it enables or protects, not because it touches the process directly.

---

## 2. Scope & Methodology

**In scope:** all hosts within the four assessed OT zones (Field/Control, Supervisory, Engineering, Monitoring). Physical field instrumentation (sensors, actuators, VFDs) downstream of the PLC is referenced conceptually but was not independently inventoried, as it exists only as simulated I/O in this lab environment.

**Criticality methodology:** each asset is scored across three dimensions, then combined into an overall ranking:

| Dimension | Definition | Scale |
|---|---|---|
| **Process Proximity** | How directly the asset's compromise can affect physical treatment operations | 1 (no path) – 5 (direct, unmediated control) |
| **Data Sensitivity** | Consequence of the asset's data being exposed, altered, or destroyed | 1 (low) – 5 (regulatory/compliance/operational record impact) |
| **Recovery Difficulty** | Operational impact of losing the asset's function, and how hard it is to work around | 1 (trivial workaround exists) – 5 (no workaround, process must stop) |

This is deliberately not a generic CVSS-style technical severity score — those measure exploitability, not consequence. A trivially exploitable asset with no path to the process (e.g., the monitoring host) is technically "vulnerable" but operationally low-stakes compared to a hardened asset that directly touches dosing or flow control. Consequence-first ranking is the ops-background differentiator this artifact exists to demonstrate.

---

## 3. Asset Inventory

| Asset | Zone | IP | Function | Key Services / Protocols |
|---|---|---|---|---|
| PLC | Field/Control | 192.168.10.100 | Programmable logic controller — direct field process control | Modbus TCP (502), EtherNet/IP (44818), S7comm (102), OpenPLC web mgmt (8080), SSH (22) |
| HMI | Supervisory | 192.168.20.100 | Operator interface; supervisory control and visualization | Apache HTTP (80), SSH (22) |
| Historian | Supervisory | 192.168.20.101 | Process data logging, trending, and retention | PostgreSQL (5432), SSH (22) |
| Monitor | Monitoring | 192.168.40.100 | Network/security monitoring collection point | Syslog (514), SSH (22) |
| Engineering Workstation | Engineering | 192.168.30.100 | Engineering access, configuration, assessment origin point | SSH (22) |

*(Assessment platform — Kali, 192.168.30.50 — is excluded from this inventory; it is assessment tooling, not a production OT asset.)*

---

## 4. Criticality Scoring

| Asset | Process Proximity | Data Sensitivity | Recovery Difficulty | Overall Rank |
|---|---|---|---|---|
| **PLC** | 5 | 3 | 5 | **Critical (1st)** |
| **Historian** | 2 | 4 | 3 | **High (2nd)** |
| **HMI** | 3 | 2 | 4 | **High (3rd)** |
| **Engineering Workstation** | 1 | 2 | 2 | **Medium (4th)** |
| **Monitor** | 1 | 1 | 2 | **Low (5th)** |

**Rationale by asset:**

**PLC — Process Proximity 5, Recovery Difficulty 5.** This is the only asset in the environment with a direct, real-time path to physical actuation — pump states, valve positions, and, in a live deployment, chemical dosing setpoints. There is no "workaround" for a compromised or unavailable PLC the way there might be for a compromised historian (an operator can log readings manually for a while) or a compromised HMI (an operator with physical panel access, where it exists, can operate locally). If the PLC's logic is altered, the physical process is altered — this is the operational reality that makes it categorically different from every other asset in the inventory, regardless of how "important" the others feel from a pure IT standpoint. Data Sensitivity is scored lower (3, not 5) deliberately: the PLC's own stored data is minimal — its risk is in what it *does*, not what it *holds*.

**Historian — Data Sensitivity 4.** The historian holds the operational record: what actually happened, over time, at the plant. This matters for two distinct reasons an IT-only assessor might undervalue: (1) regulatory/compliance defensibility — if a water quality question ever arises, historian data is the evidence trail; corrupting it quietly is arguably a worse long-term risk than a loud, obvious outage, because it can go undetected; (2) operational trending — operators use historical data to catch slow-developing problems (a pump degrading, a dosing pattern drifting) before they become acute. Its Process Proximity is low (2) because compromising the historian, by itself, does not change what the PLC or field equipment is doing right now.

**HMI — Recovery Difficulty 4.** The HMI's Process Proximity (3) reflects that it is a supervisory, not primary-control, interface — but per the segmentation work in Artifact 3, it is now the *sole authorized path* to both the PLC and the historian. That architectural role elevates its Recovery Difficulty specifically: if the HMI is unavailable, the operator loses the intended supervisory path to both downstream assets simultaneously, even though those downstream assets are themselves unaffected. This is a direct, evidenced consequence of the segmentation design and is exactly the kind of second-order effect a criticality analysis is supposed to surface.

**Engineering Workstation — uniformly low-to-moderate.** Its risk profile is almost entirely about what it can *reach*, not what it *is* — it's the origin point for legitimate administrative access and, per the RRA's threat analysis, the most plausible lateral-movement starting point for an attacker. It scores low on direct consequence dimensions but its criticality in a real assessment would rise sharply if segmentation controls (Artifact 3) were ever rolled back.

**Monitor — lowest overall, correctly.** This asset's value is entirely defensive — it observes, it doesn't act, and it doesn't hold operationally sensitive data of its own. Its low ranking here is not a statement that monitoring doesn't matter; it's a statement that *this specific host's compromise* has the least direct operational consequence of anything in the inventory, which is the correct read for a criticality ranking specifically (a separate document — the ERP — already treats monitoring's *functional* importance to detection and response, which is a different question than asset criticality).

---

## 5. How This Feeds Downstream Assessment Work

This ranking is not academic — it is the reason prior remediation work was sequenced the way it was:

- **Segmentation (Artifact 3)** prioritized closing PLC and historian exposure first, consistent with their #1 and #2 criticality rank here
- **RRA risk matrix** weighted "unauthorized PLC parameter modification" as the top risk specifically because Process Proximity and Recovery Difficulty are both maximal for that asset — the risk score wasn't assigned independently of this inventory, it was derived from it
- **ERP recovery sequencing** requires HMI integrity verification before PLC or historian reconnection — a direct consequence of HMI's elevated Recovery Difficulty score identified here, since HMI failure cascades to both downstream assets at once

A criticality ranking that doesn't visibly shape the assessments built on top of it is decoration. This one does — each downstream document can be traced back to a specific scoring rationale above.

---

## 6. Standards Alignment

This methodology aligns with the asset-management foundation IEC 62443-2-1 requires before any zone/conduit or security-level work can be meaningfully assigned, and with NIST 800-53's CM-8 (System Component Inventory) control — see the NIST 800-53/IEC 62443 crosswalk (separate artifact) for the formal control mapping.

---

## 7. Lessons Learned

**Consequence-based ranking and technical-exploitability ranking produce different priority orders, and conflating them is a real risk.** If this inventory had been scored purely on "how easy is this to attack" (an IT-security instinct), the Engineering Workstation — the most broadly reachable host — might have outranked the PLC, which is a small, narrow, protocol-specific target. That would have been the wrong prioritization for an OT environment, where the question that matters most is not "what can an attacker reach easiest" but "what happens to the physical process if they get there." This is the single clearest illustration, across this entire portfolio, of why an operations background changes an assessment's conclusions and not just its narrative framing.
