# ICS401V Concepts Linked to Water and Wastewater Operations Experience

**Author:** Moses Perodin
**Context:** 10+ years across both drinking water and wastewater operations, spanning multiple facility types — activated sludge plants, RO (reverse osmosis) units, pond systems, CSO (combined sewer overflow) handling, and municipal treatment systems. Currently operating a pretreatment anaerobic digester plant using Ignition SCADA HMI for monitoring and operational control. This document maps ICS401V cybersecurity assessment concepts to direct operational experience running the following treatment train:

- Influent EQ tank
- 2x Anaerobic digesters (methane-producing)
- Biogas flare (methane safety system)
- SBR tank (nitrification/denitrification, aeration basin)
- Effluent EQ tank
- DAF (dissolved air floatation) system — phosphorus removal, coagulation/flocculation, polishing
- Sludge recycle loop (DAF to SBR)
- Sludge holding tank
- DAF float tank (solids wasting/holding)

---

## 1. Purdue Model Zones ↔ Treatment Train Segmentation

**Operations context:**
The plant is inherently a segmented, sequential process: Influent EQ → Anaerobic Digestion → SBR → Effluent EQ → DAF → Sludge handling. Each unit process has distinct control logic, instrumentation, and operational tolerances. A failure in DAF dosing doesn't directly cascade into digester failure — the process stages are functionally isolated even though they're hydraulically connected.

**ICS401V parallel:**
The Purdue Model segments OT networks into levels (Level 0 field devices/sensors, Level 1 basic control, Level 2 supervisory control, Level 3 operations management) specifically to contain failure and limit lateral movement. Segmentation assessment asks: does a compromise at one level/zone propagate to another?

**Assessment insight:**
I ran a segmented process before I had a name for the segmentation principle. Each unit process has its own control needs and risk profile — that operational logic translates directly into evaluating whether a plant's network segmentation actually matches its process segmentation, or whether it's flat and unprotected despite the physical process being isolated.

---

## 2. Biogas Flare System = Safety-Critical OT (Highest-Stakes Linkage)

**Operations context:**
The anaerobic digesters produce methane as a byproduct of digestion. Methane is explosive within a specific concentration range in air. The biogas flare exists to safely combust excess gas rather than allow uncontrolled accumulation or venting. This is not a "nice to have" control — it is the barrier between normal operations and a genuine explosion hazard.

**ICS401V parallel:**
This maps directly to Safety Instrumented Systems (SIS) — the OT equivalent of a last-resort fail-safe layer, separate from the basic process control system. SIS compromise is treated as the most severe category of ICS attack because it removes the final layer of protection against physical harm to people and equipment. The TRITON/TRISIS malware (2017), which specifically targeted Schneider Electric Triconex safety controllers at a petrochemical plant, is the canonical example — the attacker's goal was to disable safety systems, not just disrupt production.

**Assessment insight:**
I operated a system where a control failure isn't measured in downtime — it's measured in explosion risk. When I assess or discuss SIS security in an ICS context, I'm not working from a textbook scenario. I understand viscerally why safety system integrity has to be treated as a separate, higher-priority category from general process control security.

---

## 3. Redundant/Recycle Streams ↔ Criticality Ranking for Risk Assessment

**Operations context:**
The DAF float tank and sludge holding tank function as buffers. If DAF underperforms temporarily, there's holding capacity that delays — but doesn't prevent — a compliance problem. The SBR-to-DAF sludge recycle loop also builds in a degree of process resilience. Compare this to the anaerobic digesters: there is no downstream buffer between digester upset and biogas handling consequences. Some failures buy hours; others buy minutes.

**ICS401V parallel:**
CISA's Risk and Resilience Assessment (RRA) methodology requires ranking assets by operational impact, not just technical severity (e.g., CVSS score). A vulnerability sitting at a buffered, redundant point in the process is a materially different risk than the same vulnerability sitting at a true single point of failure.

**Assessment insight:**
Most risk assessments performed by people without operations backgrounds default to generic criticality scoring. I know, from running the equipment, which failures are absorbed by the process and which aren't. That's the operational judgment a risk assessment needs to be credible — and it's a gap in a lot of assessment work done by cybersecurity-only professionals.

---

## 4. SBR Phase-Based Operation ↔ Context-Aware Anomaly Detection

**Operations context:**
The SBR (Sequencing Batch Reactor) doesn't run continuously — it cycles through phases: aeration (nitrification), anoxic period (denitrification), settling, and decant, on a timed and sensor-driven sequence. The same instrument reading means something different depending on which phase the tank is in. A DO (dissolved oxygen) spike during aeration is expected; the same spike during the settle phase signals a problem.

**ICS401V parallel:**
ICS anomaly detection tools (Zeek, Suricata) are only effective when tuned to a baseline of normal *operational* sequences, not just static network signatures. A command or data value that's benign during one process phase can be anomalous during another — detection has to be process-state-aware, not just protocol-aware.

**Assessment insight:**
I monitored a process where identical signals require different interpretations depending on context. That's precisely the challenge in effective ICS detection engineering — building rules that understand what the process is *supposed* to be doing at a given moment, not just what "normal" looks like in the abstract.

---

## 5. DAF Chemical Dosing ↔ Silent Setpoint Manipulation Risk

**Operations context:**
The DAF (dissolved air floatation) is our final treatment step before effluent transfer to the downstream plant. It's the last quality gate — we treat for TSS (total suspended solids), phosphorus removal, and BOD (biochemical oxygen demand) reduction through coagulation/flocculation and polishing. A manipulated or drifted dosing setpoint doesn't cause an obvious, immediate failure — it degrades treatment quality gradually. Marginal phosphorus or TSS removal shows up first as subtle parameter drift in the effluent; permit exceedances come later. If the system isn't tuned properly, we get hit on TSS levels along with phosphorus. Because this is pretreatment, our quality directly impacts the downstream plant's ability to meet their discharge permits. A silent setpoint drift here cascades downstream without obvious causation.

**ICS401V parallel:**
This is a direct operational analog to the ICS Cyber Kill Chain concept of setpoint manipulation as a stealth attack vector. Stuxnet's attack on centrifuge speeds is the reference case — the manipulation was designed to cause gradual physical degradation while feeding operators falsified "normal" readings, delaying detection. In a pretreatment context, the attack is even more insidious: the downstream plant sees the problem, not us.

**Assessment insight:**
I know what a subtle dosing drift looks like in pretreatment before it cascades into permit violations at the downstream facility — and I know how multiple parameters (TSS, phosphorus, BOD) can fail together if chemical dosing goes wrong. That operational pattern-recognition is what makes "silent setpoint manipulation" a concrete, understood threat to me rather than an abstract textbook scenario. It's directly relevant to why continuous process validation (not just perimeter security) matters in OT environments — especially in interconnected treatment chains where one facility's control failure becomes another facility's compliance problem.

---
