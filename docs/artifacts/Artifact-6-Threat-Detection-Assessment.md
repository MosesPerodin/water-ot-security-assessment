# OT Threat Detection Assessment
## Shenandoah Valley Water Authority — Network IDS Deployment and Detection Coverage Evaluation

**Assessment type:** Network intrusion detection deployment, capability verification, and detection coverage evaluation
**Platform:** Suricata 7.0.3, Emerging Threats Open ruleset (52,051 signatures loaded)
**Deployment host:** Monitoring zone host (192.168.40.100)
**Assessment period:** August 28–30, 2026
**Related artifacts:** Artifact 3 (Segmentation Assessment), Artifact 4 (Asset Inventory & Criticality), RRA, ERP

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## 1. Executive Summary

This assessment deployed a network intrusion detection system into Shenandoah Valley Water Authority's segmented OT environment and evaluated what it can and cannot detect. The deployment itself succeeded: Suricata is running, stable, and demonstrably detecting and alerting on traffic it can see.

**The more valuable result is what the assessment revealed about the limits of that visibility.** Two findings shape the conclusion, and neither was the expected outcome going in:

**First, an IDS deployed on its own interface behind a default-deny firewall can only inspect traffic the firewall already permits.** A port scan against the monitoring host produced records for exactly one of five scanned ports — the one port its own firewall allows. The other four were dropped at the hypervisor firewall before the IDS could ever see them. This is not a tuning problem; it is a structural consequence of where the sensor sits. An IDS in this position sees what got through, never what was blocked — which is precisely the traffic a security analyst most wants visibility into.

**Second, the stock Emerging Threats Open ruleset does not detect a generic SSH credential-guessing attempt.** Eight consecutive failed authentication attempts from an unauthorized source generated protocol event records but zero alerts. Verification against the loaded ruleset confirmed the mechanism: ET Open contains SSH-related signatures keyed to specific attack-tool fingerprints, but no rule that counts authentication failures and alerts on a rate threshold. Rate-based brute-force detection requires explicit threshold configuration that a default deployment does not include.

**Both findings are negative results, and both were verified rather than assumed.** A custom detection rule was deployed as a positive control and fired reliably, confirming that the capture-to-alert pipeline functions correctly end-to-end. This distinction matters: without the positive control, "no alerts fired" would have been indistinguishable from "the sensor is broken." With it, the negative findings above are demonstrable coverage gaps rather than unexplained silence.

**Consequence framing for a water utility:** a plant relying on a default IDS deployment in this configuration would have no alert-based visibility into credential-guessing against its OT hosts, and no visibility at all into traffic its firewall already rejected — including reconnaissance that indicates an attacker probing for a way in. Detection would depend entirely on someone manually reviewing flow logs, which is not a control anyone should count on during an incident.

---

## 2. Scope and Methodology

**In scope:** deployment and configuration of Suricata on the Monitoring zone host; verification of detection capability; evaluation of stock ruleset coverage against two attack patterns relevant to this environment's documented risks.

**Out of scope for this assessment:** cross-zone traffic inspection via port mirroring. Mirroring was attempted and is documented separately (see Section 7 and the accompanying investigation handoff); it was not completed, and this assessment does not claim cross-zone detection coverage.

**Test methodology** follows the same discipline established in Artifact 3:

- Every test uses a documented before-state baseline (log line counts, alert counts) captured immediately prior to the test action
- Test traffic uses `-T3` timing, consistent with the restrained-scanning methodology applied throughout this portfolio (aggressive scanning can destabilize legacy OT devices; documenting that restraint is itself part of the methodology)
- Results are reported as measured, including negative results
- Where a result could have more than one explanation, an additional test was run to distinguish between them rather than selecting the more favorable interpretation

**Attack patterns selected for testing** map directly to risks already documented in this portfolio:
1. **Port scanning / reconnaissance** — corresponds to MITRE ATT&CK for ICS T0846 (Remote System Discovery), identified in the RRA's threat analysis
2. **Credential guessing against a management interface** — corresponds to the credential-weakness class documented as Finding 1 in Artifact 3 (default credentials on the PLC web interface), and to T1694.001 / T0859 in the RRA's ATT&CK mapping

---

## 3. Deployment and Stabilization

Suricata 7.0.3 was deployed to the Monitoring zone host with the Emerging Threats Open ruleset (52,051 signatures active at time of testing).

**Finding 6.1 — Stock configuration referenced a network interface that does not exist on this platform, causing continuous service failure.**

The distribution's default `suricata.yaml` specifies `eth0` as the af-packet capture interface. The deployment host uses predictable network interface naming, where the interface is `ens18`. Suricata loaded its full ruleset successfully on each start, then failed when attempting to open a capture socket on a nonexistent device, and was restarted automatically by its service manager — accumulating over 14,000 restart cycles.

**Why this matters beyond the immediate fix:** the service reported `active` to standard status checks throughout, because each check happened to catch it mid-restart. An operator monitoring service health through routine status polling would have seen a running IDS. In practice there was no packet inspection occurring at all, and no alerts were possible. **A monitoring system that fails silently while reporting healthy is worse than one that fails visibly**, because it produces false confidence in a control that is not operating.

**Remediation:** capture interface corrected to `ens18`. Verified stable operation with a restart count of zero over a multi-day period.

**Finding 6.2 — Custom detection rules were silently ignored due to a path resolution mismatch.**

A custom rule file placed in `/etc/suricata/rules/` was not loaded, despite being correctly referenced in the configuration's `rule-files` list and containing valid syntax. The configuration's `default-rule-path` resolves relative paths against `/var/lib/suricata/rules`, not `/etc/suricata/rules`. Suricata reported no error for the missing file during normal operation; the mismatch surfaced only when the configuration was explicitly validated in test mode.

**Why this matters:** an analyst who writes a detection rule, sees no error, and observes the service running would reasonably conclude the rule is active. It was not. This is the same class of failure as Finding 6.1 and as **Finding 3 in Artifact 3** (firewall configuration present in config files but never enforced by the platform): *configuration that exists is not the same as configuration that is in effect.* This portfolio has now encountered that distinction three separate times, in three different subsystems. It should be treated as a standing verification requirement, not a series of coincidences.

**Remediation:** rule file relocated to the path the configuration actually resolves. Verified by confirming the loaded rule count incremented from 52,051 to 52,052.

---

## 4. Detection Capability Verification (Positive Control)

Before drawing any conclusion from tests that produced no alerts, the alerting pipeline itself was verified.

A custom rule was deployed to alert on SSH connections to the protected network:

```
alert ssh any any -> $HOME_NET 22 (msg:"LOCAL TEST - SSH connection detected"; sid:9000001; rev:1;)
```

**Result: the rule fired reliably**, generating alerts in both `eve.json` and `fast.log` across multiple independent connection attempts, with correct source and destination attribution.

Representative alert output:

```
08/30/2026-22:02:30.825517  [**] [1:9000001:1] LOCAL TEST - SSH connection detected [**]
[Classification: (null)] [Priority: 3] {TCP} 192.168.30.50:59672 -> 192.168.40.100:22
```

**What this establishes:** packet capture, protocol parsing (SSH was correctly identified as the application protocol, with client and server version strings extracted), rule evaluation, and alert logging to both output formats all function correctly.

**Why this step was necessary:** the two coverage tests in Section 5 both produced zero alerts. Without this control, those results would be ambiguous — a genuine coverage gap and a non-functioning sensor produce identical output. This test removes that ambiguity. Any assessment reporting "no alerts detected" without establishing that alerts *can* be detected is reporting an untested assumption.

---

## 5. Detection Coverage Findings

### Finding 6.3 — Sensor placement behind a default-deny firewall structurally limits visibility to permitted traffic (High significance)

**Test:** a five-port scan (22, 80, 443, 514, 8080) was directed at the monitoring host from an unauthorized source in the Engineering zone.

**Scan result** (consistent with the segmented architecture documented in Artifact 3):

| Port | State |
|---|---|
| 22 (SSH) | open |
| 80 (HTTP) | filtered |
| 443 (HTTPS) | filtered |
| 514 (syslog) | filtered |
| 8080 (HTTP-alt) | filtered |

**IDS result:** protocol event records appeared for port 22 only. Ports 80, 443, 514, and 8080 produced no records of any kind — not merely no alerts, but no evidence the traffic existed.

**Mechanism, confirmed via live firewall counters:** the host's hypervisor-level firewall enforces default-deny, permitting only SSH from the Engineering subnet and syslog from the OT subnets. Counters captured immediately after the scan showed the catch-all drop rules absorbing the scan packets directed at non-permitted ports. Suricata operates inside the guest, downstream of that enforcement point — the dropped packets were never available for inspection.

**Consequence framing:** this is the security equivalent of a plant camera positioned inside a locked room. It sees whoever comes through the door; it has no view of anyone testing the lock. In an OT environment, **reconnaissance against a control host is itself a high-value indicator** — it typically precedes an intrusion attempt and is one of the earliest opportunities to detect an adversary. An IDS placed as this one is cannot provide that indicator.

**This finding is the technical argument for network traffic mirroring**, and it is now demonstrated with evidence rather than asserted as a design preference. A sensor needs a vantage point that sees traffic *before* filtering — a mirrored feed from zone boundaries — to observe attempts as well as successes. That work was attempted during this assessment and did not complete (Section 7).

### Finding 6.4 — Stock ET Open ruleset does not detect generic SSH credential-guessing by authentication-failure rate (High significance)

**Test:** eight consecutive SSH authentication attempts with invalid credentials, from an unauthorized source in the Engineering zone against the monitoring host — a simplified credential-guessing pattern.

**Result:**

| Metric | Baseline | Post-test | Delta |
|---|---|---|---|
| Total log records | 9,930 | 9,944 | +14 |
| SSH protocol events | 22 | 31 | +9 |
| **Alert events** | **0** | **0** | **0** |

All eight attempts reached the target's SSH service, failed authentication, and were logged by Suricata as SSH protocol events with correct source attribution. **The traffic was inspected. No rule matched.**

**Mechanism, verified against the loaded ruleset rather than assumed:** the full ET Open set was active (52,051 signatures — not an empty or partially-loaded ruleset). Examination of SSH-related signatures found rules keyed to specific attack-tool fingerprints and client software banners, but no signature that tracks authentication failure frequency and alerts on a threshold. Rate-based detection of this kind requires explicit `threshold.config` entries or `detection_filter` rules, neither of which is present in a default deployment.

**Honest scoping of this finding:** eight sequential attempts, each establishing and tearing down its own connection, is a modest test volume. A properly configured rate-based rule would typically trigger on a tighter, higher-volume burst. This test does not establish that rate-based detection *would fail if configured* — it establishes that **no such detection exists out of the box**, which is the claim being made.

**Consequence framing:** credential weakness is the single most significant vulnerability class documented in this portfolio — Finding 1 of Artifact 3 was live default credentials on the PLC management interface, and the RRA scored the associated risk highest of any in the environment. A utility deploying a default IDS configuration and assuming it covers credential attacks would be wrong, and would have no alert-based indication of an ongoing credential-guessing attempt against its OT assets. **The control most needed against the highest-scored risk is the one the default configuration does not provide.**

---

## 6. Assessment of Detection Posture

| Capability | Status | Evidence |
|---|---|---|
| IDS deployed and operational | ✅ Verified | Stable service, zero restarts over multi-day period |
| Packet capture functioning | ✅ Verified | Protocol events logged with correct attribution |
| Protocol parsing (SSH) | ✅ Verified | Client/server version strings correctly extracted |
| Rule evaluation and alerting | ✅ Verified | Custom rule fired reliably across independent tests |
| Alert output to `fast.log` and `eve.json` | ✅ Verified | Alerts present in both formats |
| Reconnaissance/scan detection | ❌ Not achieved | Blocked traffic never reaches sensor (Finding 6.3) |
| Credential-attack detection | ❌ Not achieved | No matching signature in stock ruleset (Finding 6.4) |
| Cross-zone conduit inspection | ⚠️ Not achieved | Mirroring unresolved (Section 7) |
| OT protocol inspection (Modbus, S7comm, EtherNet/IP) | ⚠️ Not evaluated | Dependent on cross-zone visibility |

**Overall:** the sensor works. Its coverage, in this configuration, does not extend to either of the two attack patterns most relevant to this environment's documented risks. That is the finding — not a deployment failure, but an accurate characterization of what a default IDS deployment does and does not provide.

---

## 7. Traffic Mirroring — Attempted, Unresolved

Extending sensor visibility to cross-zone conduit traffic (PLC↔HMI Modbus, HMI↔Historian PostgreSQL) requires mirroring traffic from the zone-boundary interfaces to the monitoring host. This was attempted and did not complete.

**Summary of what was found:**

| Fault | Status |
|---|---|
| Mirror destination targeting a bridge device caused delivery failure via MAC forwarding | Resolved |
| `tc filter replace` stacking filters rather than replacing them, producing a false-positive "working" result | Resolved |
| Configuration state drift between sessions | Observed, cause not determined |
| Mirrored frames confirmed sent by kernel counters but not observed arriving at the injection point | **Unresolved** |
| Guest interface not in promiscuous mode despite configuration default | Confirmed, secondary, unresolved |

Full elimination methodology, evidence, ruled-out hypotheses, and the identified next diagnostic step are documented in **`Artifact-6-Mirroring-Investigation-Handoff.md`**.

**Disposition:** paused after four distinct faults, each resolved fault revealing another beneath it. This follows the same disposition applied to the PLC-to-historian egress anomaly in Artifact 3, Appendix A: a documented, systematically-eliminated unresolved finding is legitimate assessment output. **This assessment does not claim cross-zone detection coverage**, and Finding 6.3 above stands as the demonstrated argument for why that coverage matters.

One correction was made to the investigation record after initial commit: a "zero delivery" claim was found to be more precisely "zero delivery within the controlled test window measured," after later log review found a small number of records likely delivered during an earlier misconfigured window. The correction is documented in the handoff rather than silently amended.

---

## 8. Recommendations

Ordered by consequence, consistent with the RRA's prioritization approach.

**1 — Configure rate-based authentication-failure detection.** Finding 6.4 identifies a gap directly against the highest-scored risk class in the RRA. This requires explicit threshold or detection-filter configuration; it is not provided by the stock ruleset. Low effort, high value.

**2 — Establish sensor visibility ahead of firewall enforcement.** Finding 6.3 shows the current sensor placement cannot observe blocked traffic, including reconnaissance. Resolving the mirroring work (Section 7) or repositioning the sensor is the prerequisite for any meaningful cross-zone or attempt-based detection.

**3 — Add OT-protocol-aware detection once cross-zone visibility exists.** ET Open is a general-purpose IT ruleset. Modbus, S7comm, and EtherNet/IP function-code inspection — the protocols carrying actual process control in this environment — require OT-specific rulesets or protocol analyzers. This is sequenced after Recommendation 2 because it is untestable without cross-zone traffic.

**4 — Institute configuration-verification as standing practice.** Findings 6.1 and 6.2, together with Finding 3 from Artifact 3, establish a pattern: this environment has produced three separate instances of configuration that appeared correct but was not in effect. Verification should confirm *operational state* (rule counts, enforcement counters, service restart counts), not the presence of configuration files.

**5 — Configure log rotation and review host resource allocation.** The monitoring host operates with limited headroom (approximately 1.3 GB disk free, under 600 MB available memory, with swap in use). Event logs already exceed 50 MB. Increasing inspected traffic volume — the goal of Recommendation 2 — will accelerate log growth against constrained resources.

---

## 9. Lessons Learned

**Verify the detection pipeline before interpreting the absence of detections.** Two of this assessment's substantive findings are negative results. Without the positive control in Section 4, both would have been unsupportable — "no alerts" is equally consistent with good coverage of benign traffic, a coverage gap, and a completely broken sensor. **The control test is what converts an absence of evidence into evidence of absence.** This is a general principle for detection assessment work, not a detail specific to this deployment.

**A sensor's value is bounded by its vantage point, and vantage point is an architecture decision, not a tuning decision.** No amount of rule refinement would have let this sensor see the four scanned ports its firewall dropped. Detection coverage is determined first by where the sensor sits and second by what rules it runs — and the first constraint cannot be fixed by adjusting the second.

**Default configurations are a starting point, not a control.** The stock deployment shipped with a nonexistent capture interface, a rule path that silently ignores custom rules, and no coverage for credential-guessing attacks. Each was individually minor and easily fixed once identified. Collectively they meant a utility deploying this configuration and considering the task complete would have an IDS that appeared healthy, loaded no custom detections, and covered neither of the attack patterns most relevant to its environment.

**Correcting a false-positive conclusion mid-assessment is part of the work.** An intermediate result in the mirroring investigation was initially read as confirming successful delivery; further testing showed the evidence came from a different, misconfigured path. That correction is recorded in the investigation handoff rather than quietly dropped. An assessment methodology that cannot catch and correct its own errors is not a methodology — and the record of having done so is stronger evidence of rigor than an unbroken string of clean results would be.

---

## 10. Standards Alignment

| Framework | Reference | Relevance |
|---|---|---|
| NIST 800-53 | SI-4 (System Monitoring) | Directly addressed; this assessment evaluates monitoring capability and documents its coverage limits |
| NIST 800-53 | AU-6 (Audit Review, Analysis, Reporting) | Partially addressed; log generation verified, automated analysis not yet configured |
| IEC 62443-3-3 | FR6 — Timely Response to Events | Detection is a prerequisite for the response procedures defined in the ERP |
| NIST 800-82 | ICS monitoring guidance | Sensor placement and OT protocol visibility considerations reflected in Recommendations 2–3 |
| MITRE ATT&CK for ICS | T0846 (Remote System Discovery) | Tested; not detected in current configuration — Finding 6.3 |
| EPA Checklist (817-B-23-001) | 2.T (security log collection) | Advanced from partial toward implemented; alerting gaps documented in Findings 6.3–6.4 |

See the NIST 800-53 / IEC 62443 crosswalk artifact for full control mapping across the portfolio.
