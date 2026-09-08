# MITRE ATT&CK for ICS Incident Mapping
## Case Study: Oldsmar, Florida Water Treatment Facility (February 2021)

**Assessment type:** Blue-team retrospective — historical incident mapped to MITRE ATT&CK for ICS, with defensive lessons applied to Shenandoah Valley Water Authority's own architecture
**Purpose:** demonstrate threat-modeling capability using a real, sector-relevant incident, and connect its lessons directly to controls already implemented in this portfolio (Artifact 3)

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement. (The Oldsmar, Florida incident analyzed here is a real, publicly documented event; only the connection to Shenandoah Valley is fictional.)

---

## 1. Executive Summary

On February 5, 2021, an operator at the Bruce T. Haddock Water Treatment Plant in Oldsmar, Florida observed a remote user access the plant's SCADA workstation and increase the sodium hydroxide (lye) setpoint from 100 ppm to approximately 11,100 ppm — a roughly 100x increase in a chemical used to control water acidity. The operator reversed the change within minutes. Public water supply was never at risk; officials stated any effect would have taken 24–36 hours to reach customers, and automated pH alarms would have caught it independently.

**This case is included specifically because of, not despite, a complication most public write-ups skip:** more than two years after the incident, the FBI stated it could not confirm the event was a targeted cyber intrusion at all, and Oldsmar's own city manager at the time described it as a "nonevent," suggesting it may have been an operator's own mistake rather than an attack. This is disclosed here plainly, not glossed over — an assessor who cites a headline without checking whether it held up under investigation is doing exactly the kind of un-rigorous work this portfolio exists to demonstrate the opposite of.

**Why this case study still has value regardless of that ambiguity:** the technical environment described in every contemporaneous advisory — Windows 7, TeamViewer with a shared password, no firewall, direct internet exposure — is a real, well-documented, and depressingly common water-sector risk profile, independent of whether this specific event was an attack or an accident. The ATT&CK mapping below maps the *reported* attack chain as public advisories described it, which remains a legitimate and useful reference model for the access pattern this kind of environment is vulnerable to, whether the Oldsmar incident specifically was that pattern in action or not.

---

## 2. Incident Timeline (as reported)

| Time (Feb 5, 2021) | Event |
|---|---|
| ~8:00 AM | Operator observes the mouse cursor moving unexpectedly on the SCADA workstation; assumes it is a supervisor performing routine remote monitoring (not unusual at this facility) and does not intervene |
| ~1:30 PM | Operator observes a second remote session; various software functions opened over 3–5 minutes |
| ~1:30 PM (same session) | Sodium hydroxide setpoint changed from 100 ppm to ~11,100 ppm via the HMI software |
| Immediately after | Operator observes the change, manually reverts the setpoint to 100 ppm |
| Same day | Facility supervisor notified; remote access subsequently disabled |

**Technical conditions confirmed by FBI/state advisories** (regardless of intent question): all SCADA-connected computers ran 32-bit Windows 7 (unsupported since January 2020), all had TeamViewer installed for remote monitoring/troubleshooting, all TeamViewer instances shared a single common password among staff, and the system was directly internet-connected with no firewall.

---

## 3. MITRE ATT&CK for ICS Mapping

Mapped against the publicly reported access pattern. Technique IDs verified against the current ATT&CK for ICS matrix.

| Stage | ATT&CK for ICS Technique | Application to Oldsmar |
|---|---|---|
| Initial Access | **T0886 — Remote Services** | TeamViewer, an internet-facing remote access tool, was the reported entry vector — legitimate software used for illegitimate access, not a software exploit |
| Initial Access (enabling condition) | **T1078-class Valid Accounts** *(IT-ATT&CK; ICS matrix treats credential reuse as a Persistence/Initial Access enabler under the same conceptual umbrella as T0886)* | A single TeamViewer password shared across all staff meant one credential compromise (however it occurred) granted access indistinguishable from legitimate staff use |
| Discovery | **T0846 — Remote System Discovery** *(implicit)* | Reported behavior (moving between software functions over several minutes before the setpoint change) is consistent with an operator or intruder navigating an unfamiliar HMI to locate the correct control |
| Impact | **T0836 — Modify Parameter** | The direct action: sodium hydroxide setpoint changed via the HMI software interface |
| (Would-be) Impact | **T0806 — Loss of Safety** *(prevented)* | Had the change gone undetected, the consequence pathway ran through water treatment chemistry to a public health outcome — this is the "worst case" the incident is remembered for, even though redundant pH alarms meant this specific pathway had a second layer of defense that was never actually tested by events |

**What this chain does *not* include, and why that matters:** there is no exploitation technique in this mapping — no malware, no vulnerability exploit, no privilege escalation. Every step, if it happened as an attack, used legitimate remote-access software and a legitimate (shared) credential. This is the single most important pattern to take from this case: **the compromise, if real, required no technical sophistication, only weak identity and access management layered on top of an internet-exposed remote access tool.** This exact pattern — not a specific exploit — is what a defensive architecture needs to be built against.

---

## 4. Blue-Team Assessment: How Shenandoah Valley's Architecture Would Fare Against This Pattern

This is not a hypothetical exercise — it is a direct comparison against the controls documented in Artifact 3, RRA, and the AWWA assessment.

| Oldsmar Condition | Shenandoah Valley Status | Assessment |
|---|---|---|
| SCADA workstation directly internet-connected | OT-zone hosts have no internet egress by design (confirmed architectural constraint) | **Materially stronger.** The entry vector itself (internet-facing remote tool) doesn't exist in this architecture |
| Remote access via consumer remote-desktop software (TeamViewer) | Remote access is SSH, key-based authentication, through defined conduits only | **Materially stronger.** No consumer remote-access tool is in the environment; access requires a private key, not a shared password |
| Shared password across all staff for remote access | Not directly assessed for this environment (single-operator lab), but AWWA Section 3 already flags no formal credential policy exists across OT assets | **Gap, honestly acknowledged.** The specific failure mode (shared credentials) has not been tested here, and the AWWA assessment already scores this area Repeatable, not Defined — this case study reinforces why that gap matters, using a concrete real-world precedent rather than an abstract control description |
| No firewall on SCADA-connected systems | Default-deny, zone-based firewall enforced and independently verified (Artifact 3) | **Materially stronger.** This is the single largest architectural difference between the two environments |
| Unsupported, unpatched OS (Windows 7) | Uniform, current OpenSSH baseline across OT hosts at assessment time (RRA Section 1) | **Materially stronger**, though this reflects a lab built fresh in 2026, not an aging facility with years of deferred maintenance — a real utility's patch posture would need independent verification, not assumed from a lab comparison |
| No network segmentation between remote access point and control function | Artifact 3's entire scope — segmented, conduit-restricted, independently verified | **Materially stronger** — this is the specific gap Artifact 3 was built to close |

**Net assessment:** the architectural work already documented in this portfolio would have closed or materially narrowed nearly every technical condition that made the Oldsmar pattern possible, with one explicit exception — credential policy across all OT hosts is not yet formalized, a gap this case study makes concrete rather than abstract.

---

## 5. Lessons Learned

**Verify the incident before building an assessment around it, not after.** This case study's most important methodological lesson isn't about Oldsmar's technical failures — it's that the widely-repeated "hacker tried to poison a town's water" narrative was, by the FBI's own later account, never confirmed as a targeted intrusion. An assessor who treats every widely-reported incident as settled fact risks building conclusions on a foundation that doesn't hold. The correct practice — checking primary sources, noting where investigators walked claims back, and citing the ambiguity rather than the cleaner story — is itself professional-grade work, arguably more valuable to demonstrate than a clean incident-response narrative would have been.

**The controls that matter here are boring, not exotic.** Nothing about the *reported* Oldsmar chain required nation-state tradecraft. A shared password, an internet-facing remote tool, and no network segmentation would be sufficient (again, if the reported chain is accurate) to reach a safety-relevant control point. This is a useful corrective against threat-modeling exercises that overweight sophisticated adversary techniques (T0866-class exploitation, custom malware) relative to identity hygiene and network architecture — the fundamentals this entire portfolio has been built around are precisely the fundamentals this case turns on.

**Redundant, independent safety controls are what "resilience" actually looks like in practice.** Even under the worst-case reading of this incident, the automated pH alarm system was a second, independent layer that never depended on the operator's vigilance. Shenandoah Valley's own segmentation work (Artifact 3) is analogous — it doesn't assume any single control (a credential, a firewall rule) will hold forever; it's designed so that a failure in one layer doesn't cascade into a physical-process consequence on its own.
