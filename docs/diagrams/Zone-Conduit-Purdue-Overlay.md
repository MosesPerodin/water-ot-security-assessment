# Zone / Conduit Diagram — Purdue Model Overlay

**Scope:** the OT network's **after-remediation** state — default-deny zone
boundaries with explicitly authorized conduits, as enforced by the Proxmox
host-level firewall (`pve-firewall`) and documented in
[Artifact 3](../artifacts/Artifact-3-Segmentation-Assessment.md).

**Source of truth:** the per-VM `.fw` copies in
[`configs/firewall/network-segmentation/`](../../configs/firewall/network-segmentation/),
re-exported from live host state on 2026-08-30, and the live-state capture in
[`../project-knowledge/LAB-TOPOLOGY.md`](../project-knowledge/LAB-TOPOLOGY.md)
(section "Proxmox firewall configuration"). This diagram is a reading aid, not an
independent authority.

---

## Diagram

```mermaid
flowchart TB
    subgraph MGMT["Management - 192.168.1.0/24 (vmbr0)"]
        PVE["Proxmox host<br/>192.168.1.250"]
    end

    subgraph ENG["Engineering - Purdue Level 3 - 192.168.30.0/24 (vmbr30)"]
        EWS["workstation-eng<br/>192.168.30.100"]
        KALI["kali-attack<br/>192.168.30.50<br/>(assessment vantage point)"]
    end

    subgraph SUP["Supervisory / HMI - Purdue Level 2 - 192.168.20.0/24 (vmbr20)"]
        HMI["hmi-scada / Apache<br/>192.168.20.100"]
        HIST["historian-db / PostgreSQL<br/>192.168.20.101"]
    end

    subgraph FIELD["Field / Control - Purdue Level 1 - 192.168.10.0/24 (vmbr10)"]
        PLC["plc-controller / OpenPLC<br/>192.168.10.100"]
    end

    subgraph MON["Monitoring - Purdue Level 3 (security) - 192.168.40.0/24 (vmbr40)"]
        SUR["monitor-wireshark / Suricata 7.0.3<br/>192.168.40.100"]
    end

    %% --- Operational conduits ---
    HMI -- "Modbus TCP / 502 (supervisory control)" --> PLC
    HMI -- "PostgreSQL / 5432 (process data logging)" --> HIST
    PLC -. "5432 - permitted but non-functional (egress anomaly, Artifact 3 App. A); relayed via HMI" .-> HIST

    %% --- Telemetry conduit (syslog) ---
    PLC -- "syslog / 514" --> SUR
    HMI -- "syslog / 514" --> SUR
    HIST -- "syslog / 514" --> SUR
    EWS -- "syslog / 514" --> SUR

    %% --- Administrative access ---
    EWS -- "SSH / 22 (admin)" --> PLC
    EWS -- "SSH / 22 (admin)" --> HMI
    EWS -- "SSH / 22 (admin)" --> HIST
    EWS -- "SSH / 22 (admin)" --> SUR
    PVE -. "SSH / 22 via zone gateway (.1)" .-> SUP
    PVE -. "SSH / 22 via zone gateway (.1)" .-> FIELD
```

Every zone boundary is **default-deny inbound** (`policy_in: DROP`). Solid
arrows are the only cross-zone flows the firewall permits; the dashed
PLC&rarr;historian arrow is authorized in policy but does not pass traffic
(see notes). Anything not drawn is blocked.

---

## Authorized conduits

| # | Source | Destination | Port / protocol | Purpose | Policy (live) |
|---|---|---|---|---|---|
| 1 | HMI `192.168.20.100` | PLC `192.168.10.100` | TCP 502 / Modbus | Supervisory control | `101.fw` |
| 2 | HMI `192.168.20.100` | Historian `192.168.20.101` | TCP 5432 / PostgreSQL | Process data logging (added during remediation) | `102.fw` |
| 3 | PLC `192.168.10.100` | Historian `192.168.20.101` | TCP 5432 / PostgreSQL | *Intended* direct logging — **permitted in `102.fw` but non-functional** (egress anomaly, Artifact 3 Appendix A). Data is relayed via conduit 2. | `102.fw` |
| 4 | All OT zones `10/20/30/40.0/24` | Monitor `192.168.40.100` | TCP 514 / syslog | Log / telemetry shipping | `104.fw` |
| 5 | Jumpbox `192.168.30.10` | PLC, HMI, Historian, Monitor | TCP 22 / SSH | Administrative access — narrowed from the full Engineering subnet (`192.168.30.0/24`) on 2026-09-27 | `100/101/102/104.fw` |
| 6 | Zone gateway `<zone>.1` (Proxmox host) | PLC `.10.1`, HMI + Historian `.20.1` | TCP 22 / SSH | Host-originated management (Proxmox sources from the bridge-local gateway IP, not the management IP) | `100/101/102.fw` |
| 7 | Management `192.168.1.0/24` | Proxmox host `192.168.1.250` | TCP 8006 / 22 | Hypervisor GUI + SSH | `cluster.fw` |
| 8 | `192.168.10.0/24`, `192.168.20.0/24` | Zone gateway `.1` | ICMP | Gateway reachability check | `cluster.fw` |

The Monitor host (`104.fw`) has **no** gateway-SSH rule — it is administered
over SSH from jumpbox only (`ssh -J pve,ubuntu@192.168.30.10 ubuntu@192.168.40.100`) — Kali is not an administrative relay and cannot reach Monitor under current policy.

---

## Purdue Model overlay

| Purdue level | Function | Lab zone (bridge) | Hosts |
|---|---|---|---|
| Level 1 | Basic control — PLC / field logic | Field / Control (`vmbr10`) | `plc-controller` (OpenPLC) |
| Level 2 | Supervisory control, HMI, historian | Supervisory / HMI (`vmbr20`) | `hmi-scada`, `historian-db` |
| Level 3 | Operations / engineering | Engineering (`vmbr30`) | `workstation-eng`, `kali-attack` |
| Level 3 (security) | Monitoring, logging, detection | Monitoring (`vmbr40`) | `monitor-wireshark` (Suricata) |
| — | Hypervisor / out-of-band management | Management (`vmbr0`) | Proxmox host |

The remediation aligned the network to IEC 62443's zones-and-conduits model and
the Purdue separation of Level 1 (Field) from Level 2 (Supervisory), restricting
Level 3 (Engineering) to administrative access only rather than the unrestricted
operational reach it had by default (Artifact 3 §9).

---

## Where this lab diverges from a real plant

Stated explicitly, per assessment methodology — the diagram models a
representative environment, not a claim of a real utility network.

- **No Industrial DMZ (Purdue Level 3.5).** There is no brokered DMZ between the
  Supervisory and Engineering zones; cross-zone flow is mediated only by the
  host-level firewall. An IDMZ is the standard next control and is noted as a
  future improvement in the segmentation case study.
- **No Level 0.** Field instrumentation (sensors, actuators, I/O networks) is not
  modeled; the PLC is the lowest level represented.
- **Single PLC.** One controller stands in for what would be multiple PLCs/RTUs
  across process areas.
- **Monitoring is host-addressed only.** Cross-zone conduit traffic is not
  mirrored to the Monitor (port mirroring paused — see
  [Artifact 6 Mirroring Investigation Handoff](../artifacts/Artifact-6-Mirroring-Investigation-Handoff.md)).
  Suricata currently inspects traffic addressed to the Monitor plus the syslog
  feed (conduit 4), not the HMI&harr;PLC or HMI&harr;historian conduits
  (Finding 6.3).
- **HMI web interface is intra-zone and unencrypted.** Apache on `192.168.20.100`
  serves HTTP with no TLS (Finding 4, open) and has no cross-zone allow rule —
  operator browser access is within the Supervisory zone only.

---

## Policy-file sync history

The `.fw` files under `configs/firewall/network-segmentation/` were first
committed 2026-08-19 and drifted from the deployed policy over the Aug 25–30
assessment window: `102` gained the HMI→5432 conduit and a gateway-SSH rule,
`101` gained a gateway-SSH rule, and `100` was never committed at all.

**Re-synced 2026-08-30** — the four files (`100/101/102/104`) now match live host
state and `100-hmi-policy.fw` was added. See
[`configs/firewall/network-segmentation/README.md`](../../configs/firewall/network-segmentation/README.md).

**Still open for the 5.4 pass:** Artifact 3 §5 says the historian's 5432 is
"permitted only from the HMI's IP (192.168.20.100)" and lists PLC→Historian as
"deliberately not implemented," but the deployed `102.fw` permits 5432 from
*both* the PLC and the HMI — the PLC rule is present but inert (0 packets) because
of the egress anomaly. The wording should be reconciled with the deployed rule
set. Separately, the committed connectivity evidence in
`docs/case-studies/001-network-segmentation/evidence/after-validation/` is the
Aug 19 set and shows the pre-conduit state (PLC→5432 worked, HMI→5432 blocked) —
the inverse of the current live state; it needs re-capture or a dated annotation.

---

## How to read the diagram

- **Boxes** are hosts; **panels** are zones, labeled with their Purdue level,
  subnet, and Proxmox bridge.
- **Solid arrows** are cross-zone flows the firewall actively permits, labeled
  with port and purpose.
- **Dashed arrows** are authorized-but-not-carrying-traffic (conduit 3) or
  host-management paths.
- Direction is the **connection initiator** &rarr; the **listener**.
- Absence of an arrow means the traffic is dropped by the destination zone's
  default-deny inbound policy.
