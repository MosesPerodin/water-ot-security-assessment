# Port Mirroring Investigation — Artifact 6 (Suricata Threat Detection)
## Shenandoah Valley Water Authority — Troubleshooting Handoff

**Date:** August 28–30, 2026
**Objective:** configure traffic mirroring from the three OT zone-boundary ports (fwpr101p0/PLC, fwpr100p0/HMI, fwpr102p0/Historian) to the Monitor VM (VM104) so Suricata can inspect cross-zone conduit traffic, not just traffic addressed directly to itself.
**Status at time of writing:** paused. One fault fully resolved and confirmed. A second, distinct fault identified and diagnosed to the host/guest boundary, but not yet root-caused. Decision made to pursue an alternate path for Artifact 6's evidence rather than continue this investigation indefinitely — documented here as a legitimate, partially-resolved finding, consistent with the documentation discipline applied to the Phase 3 PLC-historian egress anomaly during its own investigation (Finding 3.1, later diagnosed via a targeted toggle test).

**Fictional utility.** Shenandoah Valley Water Authority is not a real organization — it was created for this assessment portfolio. Every finding, evidence item, "Prepared for" line, and "Internal / security-sensitive" classification in this document is an artifact of this training portfolio, not a record of a real engagement.

---

## Background

Suricata 7.0.3 was successfully installed and stabilized on VM104 prior to this investigation (see separate session notes — the install itself required resolving an unrelated `firewall=1`/NAT bug, and a stock-config interface-name mismatch that was causing a 14,000+ restart crash loop). With Suricata stable and running, it had visibility only into traffic addressed directly to VM104 (its own SSH sessions, syslog). To make it useful for detecting cross-zone activity — the actual purpose of an OT-environment IDS — mirrored copies of PLC, HMI, and Historian zone-boundary traffic needed to reach VM104's interface.

---

## Fault 1 — Mirror destination pointed at a bridge, not a device (RESOLVED)

**Symptom:** `tc` filters were added mirroring egress traffic on all three `fwpr*` ports to `vmbr40` (the Monitor zone's bridge device). Filter counters showed traffic matching and being sent (`dropped 0`), but zero corresponding events ever appeared in Suricata's `eve.json`, even for scans that definitely crossed the relevant zone boundary.

**Root cause:** mirroring to a *bridge* device (`vmbr40`) subjects the mirrored frame to normal MAC-address-based bridge forwarding. The mirrored frames carried source/destination MAC addresses from the `vmbr10`/`vmbr20` zones, which do not exist in `vmbr40`'s forwarding database — so the frames were flooded or silently dropped by the bridge rather than reliably delivered to VM104's actual interface.

**Fix:** mirror destination changed from `vmbr40` to `tap104i0` — VM104's actual host-side tap device — which delivers frames directly to the guest's virtual NIC rather than relying on bridge forwarding logic.

**Confirmation:** on the first port fixed (`fwpr101p0`/PLC), a controlled before/after test showed `eve.json` events appearing on all five scanned ports (22, 102, 502, 8080, 44818) with a clean `+2` delta each — versus the prior state, which showed activity on port 22 only (later determined to be unrelated incidental SSH traffic, not mirrored scan traffic). This was treated as confirmed delivery at the time.

**Correction (see Fault 2):** this "confirmed" result was later shown to be an artifact of leftover `vmbr40`-destination flooding from before the fix was applied, not proof the `tap104i0` path itself was working. The lesson: a positive result immediately after a configuration change should be re-verified independently, not accepted at face value, especially in an environment where state has already proven unreliable.

---

## Fault 2 — `tc filter replace` silently stacks instead of replacing (RESOLVED, but caused a false positive)

**Symptom:** applying `tc filter replace ... action mirred egress mirror dev tap104i0` to switch a port's mirror destination did not remove the old `vmbr40` filter — `tc filter show` afterward displayed *both* the old and new mirror actions active simultaneously on the same port, with different `index` values (1, then 4, then 7 on repeated attempts).

**Root cause:** `tc filter replace` does not uniquely key on the match criteria used here (`u32 match u32 0 0` — effectively "match everything"), so each invocation added a new filter instance rather than replacing the prior one. This went undetected initially because grepping for the new destination (`tap104i0`) returned a match regardless of whether the old, broken destination (`vmbr40`) was also still present and still delivering (via bridge flooding) some fraction of traffic to VM104.

**Consequence:** this is very likely what produced Fault 1's apparent "confirmation" — traffic was reaching Suricata via the still-present `vmbr40` flood path, not via the newly-added `tap104i0` filter, creating a false impression that the fix worked.

**Fix:** abandoned `tc filter replace` for this purpose. Used `tc qdisc del dev <port> root` (removing the entire qdisc and all attached filters) followed by a clean `tc qdisc add` + `tc filter add`, guaranteeing exactly one filter with one destination per port.

**Confirmation:** verified via `tc filter show dev <port> | grep -c mirred` returning exactly `1` per port, and an explicit `grep vmbr40` returning zero matches — not inferred from the presence of the correct destination alone.

**Lesson for future work in this lab:** do not trust a configuration command's apparent success (or a single grep match) as confirmation of clean state. This is the same category of lesson as the `pve-firewall compile` unreliability finding from Phase 3 — verify the *actual* resulting state, not the tool's implied success.

---

## Fault 3 — State drift between sessions (OBSERVED, cause not investigated)

**Symptom:** at the start of a later verification pass, `fwpr100p0` and `fwpr102p0` were found with *two* active mirror filters each (both the old `vmbr40` and a `tap104i0` filter) — despite no commands having been run against those two ports since the prior session's `fwpr101p0`-only fix.

**Interpretation:** either a prior instruction was partially applied across more ports than intended, or some other process/session touched this configuration outside of what's captured in this investigation's command log. This was not investigated further, given the more pressing blockers, but is noted here because it reinforces the broader pattern seen throughout this project (the mysteriously-vanished MASQUERADE rules after a reboot, the deleted Aug 19 evidence files) of configuration state in this lab not being as stable as a single session's command history would suggest.

**Handling:** rather than assume the described "before" state, the actual live state was re-verified via read-only checks before any further changes were applied — the correct response to an environment with a demonstrated history of unexplained drift.

---

## Fault 4 — Mirrored frames confirmed sent, but not arriving at the injection point (UNRESOLVED — investigation paused here)

**Symptom:** with all three ports cleanly configured (single filter each, confirmed `tap104i0` destination, no `vmbr40` remnants), a three-target delivery test was run. Mirror filter counters (`tc -s filter show`) showed non-zero, incrementing packet counts on all three ports with `dropped 0`. `tap104i0`'s own TX counter incremented correspondingly. Yet a direct `tcpdump` capture on `tap104i0` itself — the actual injection point — showed **zero TCP frames** arriving from any of the three monitored hosts, across two separate test windows. One window captured 3 ARP reply frames; a second, otherwise similar window captured nothing at all, not even ARP.

**What this rules out:**
- **Filter matching failure** — ruled out. All three ports' `mirred` action counters increment correctly during test traffic, meaning the kernel *is* selecting and attempting to forward the intended packets.
- **Downstream/tap-level drops** — ruled out as measured. `tap104i0` shows `dropped 0` on its own interface statistics, and Suricata's own capture-path statistics (`kernel_drops: 0`) show no losses on its side either.
- **Bridge-scoped issue (vmbr10 vs vmbr20)** — ruled out. An initial working/non-working split (PLC port appearing to work, HMI/Historian ports not) was hypothesized as bridge-specific, but this hypothesis was later invalidated once it was shown the PLC port's apparent success was itself a false positive (see Fault 2's consequence). All three ports, on both bridges, behave identically: matched and "sent" per the kernel counter, absent at the injection point.
- **Guest-side (VM104-internal) unicast filtering / promiscuous mode** — a real, separate, *confirmed-present* issue (see below), but explicitly ruled out as the cause of *this specific* symptom, because the diagnostic capture was run on the **host** side of the tap device (`tcpdump -i tap104i0`, executed on pve), which observes frames before they ever reach the guest's virtual NIC. A guest-side filtering problem cannot explain frames that never arrive at the host-side capture point at all.

**Separately confirmed, real issue (Fault 5, below) that does NOT explain Fault 4:** VM104's guest network interface (`ens18`) is not in promiscuous mode, despite Suricata's configuration (`suricata.yaml`) leaving the relevant setting at its default, which is expected to enable promiscuous mode automatically. This means that *even if* Fault 4 is resolved, mirrored unicast traffic addressed to other hosts' MAC addresses would still be silently discarded by the guest NIC unless this is separately fixed. This is real and will need addressing, but it is downstream of — and does not account for — Fault 4, since Fault 4's symptom is observed on the host side, before the guest NIC is ever involved.

**Working hypothesis, not yet tested:** the one meaningful difference noted between successful and unsuccessful capture windows was that ARP (broadcast) frames were observed passing through in one window while unicast TCP frames were not, in either window. This suggests something about how QEMU/the tap device driver handles broadcast versus unicast delivery at this specific injection point — but this is speculation pending an actual test, not a conclusion. The kernel counter incrementing while the frame never appears at the tap is the central unexplained fact.

**Next diagnostic step identified, not yet run:** a capture from *inside* VM104 on `ens18` itself (`sudo tcpdump -i ens18`) would, in a single test, distinguish three remaining possibilities: (a) frames genuinely never arrive at the guest at all, (b) frames arrive but are being silently discarded due to the promiscuous-mode gap (Fault 5), or (c) frames arrive and are received correctly, in which case the fault would lie somewhere in Suricata's own packet handling rather than in the network path. This test requires root access on VM104, which was gated behind an interactive sudo password not available in this investigation's automated tooling — it would need to be run manually.

---

## Fault 5 — VM104's guest NIC is not in promiscuous mode despite Suricata's expected default (CONFIRMED, unresolved, secondary priority)

**Symptom:** `ip link show ens18` on VM104 shows flags `BROADCAST,MULTICAST,UP,LOWER_UP` — no `PROMISC` flag present.

**Context:** Suricata's `suricata.yaml` has the `disable-promisc` setting commented out (at its default), which per Suricata's own documented behavior should result in the capture interface being placed into promiscuous mode automatically when Suricata opens its af-packet capture socket. Suricata has been running for multiple days with zero restarts and is actively capturing packets (confirmed via its own `kernel_packets` statistics counter), yet the promiscuous flag is absent.

**Why this matters:** even once Fault 4 (frames not reaching the tap/guest at all) is resolved, mirrored traffic between other hosts (e.g., HMI-to-PLC Modbus traffic) will carry MAC addresses belonging to neither VM104 nor a broadcast address. A non-promiscuous NIC discards such frames at the hardware/driver level before they ever reach Suricata's capture logic — meaning this fault, left unaddressed, would silently limit detection to only broadcast traffic and traffic explicitly addressed to VM104 itself.

**Not yet investigated:** why Suricata's expected default behavior isn't taking effect on this specific interface/environment. Possible directions for a future session: explicit `ip link set ens18 promisc on` (a manual, non-persistent workaround), checking whether af-packet mode requires an explicit `checksum-checks` or capture-mode setting beyond the default, or checking whether this is a known interaction with virtio-net interfaces under QEMU specifically.

---

## Resource Constraints Observed During This Investigation

VM104 is running close to its provisioned limits, which should inform how aggressively mirroring is re-attempted in any future session:

| Metric | Value at last check |
|---|---|
| Disk free | 1.3 GB (84% used) |
| RAM available | ~576–585 MB |
| Swap in use | 114 MB (was near-zero at session start) |
| Suricata RSS | ~1.18 GB |
| eve.json size | ~53.8 MB |
| stats.log size | ~37.3 MB |

Resources remained stable throughout this investigation's test traffic (no further swap growth observed), but the appearance of any swap usage at all — where there was none before — suggests limited headroom. Before resuming this work, consider whether VM104 needs more allocated RAM, or whether log rotation/retention needs to be configured before generating larger volumes of test traffic.

---

## Decision and Path Forward

Given the pattern of this investigation — each resolved fault revealing another distinct fault beneath it, now four layers deep (destination-to-bridge, filter-stacking, state-drift, and the current unresolved host-injection-point fault) with no clear indication of how many more layers remain — the decision was made to **pause the mirroring investigation** rather than continue indefinitely, and pursue an alternate path to complete Artifact 6's evidence requirements.

The same discipline was applied to the Phase 3 PLC-to-historian egress anomaly (Finding 3.1, Artifact 3 Appendix A): after systematically eliminating fourteen hypotheses without a confirmed cause, the investigation was documented as unresolved rather than closed prematurely or left unrecorded. That documentation was not a dead end — a targeted toggle test on VM104, informed by the eliminated hypotheses, later identified the root cause. The lesson generalizes: a defensible amount of systematic elimination, rigorously documented, is legitimate portfolio content whether or not the investigation concludes with a root cause in hand — and thorough documentation of what has been ruled out is often what makes a later resolution possible. The debugging discipline demonstrated here (catching and correcting a false-positive conclusion mid-investigation, ruling out hypotheses with direct evidence rather than inference, refusing to assert an unconfirmed mechanism) is arguably as valuable a demonstration of assessment methodology as a clean, immediately-resolved outcome would have been.

**Path forward for Artifact 6:** rather than continue pursuing cross-zone mirroring, evidence will be generated using traffic addressed directly to VM104's own interface — a smaller-scoped but immediately achievable demonstration of Suricata's detection capability, without depending on resolution of Faults 4 and 5. The mirroring investigation remains documented here for any future session that wants to resume it, with the specific next diagnostic step (an in-guest `tcpdump -i ens18` capture) already identified and ready to run.
