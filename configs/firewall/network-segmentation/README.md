# Network Segmentation — Proxmox Firewall Policies

These are the per-VM `pve-firewall` policy files as deployed on the Proxmox host
(`/etc/pve/firewall/<vmid>.fw`), supporting the segmentation work in
[Artifact 3](../../../docs/artifacts/Artifact-3-Segmentation-Assessment.md) and
[Case Study 001](../../../docs/case-studies/001-network-segmentation/).

| File | VM | Zone | Inbound policy |
|---|---|---|---|
| `100-hmi-policy.fw` | 100 hmi-scada | Supervisory | default-deny; SSH from Supervisory gateway and jumpbox (192.168.30.10) only |
| `101-plc-policy.fw` | 101 plc-controller | Field / Control | default-deny; Modbus/502 from HMI; SSH from Field gateway and jumpbox (192.168.30.10) only |
| `102-historian-policy.fw` | 102 historian-db | Supervisory | default-deny; PostgreSQL/5432 from PLC and HMI; SSH from Supervisory gateway and jumpbox (192.168.30.10) only |
| `104-monitor-policy.fw` | 104 monitor-wireshark | Monitoring | default-deny; syslog/514 from all four OT zones; SSH from jumpbox (192.168.30.10) only |
| `105-kali-policy.fw` | 105 kali-attack | Engineering | default-deny; SSH from the operator's management-network host (192.168.1.167) only — no gateway fallback, deliberate |
| `106-jumpbox-policy.fw` | 106 jumpbox | Engineering | default-deny; SSH from the operator's management-network host (192.168.1.167) and the Engineering gateway (192.168.30.1) — the administrative chokepoint all other Engineering-zone access now routes through |

The `192.168.<zone>.1` SSH allow rules exist because the Proxmox host sources
connections to its guests from the bridge-local gateway IP, not the management
IP.

As of 2026-09-27, administrative SSH from Engineering was narrowed from the
full `192.168.30.0/24` subnet to jumpbox (`192.168.30.10`) specifically. Jumpbox
is itself restricted to the operator's own management host (`192.168.1.167/32`)
plus the Engineering gateway rule, since it is the intended chokepoint other
Engineering-zone administration passes through. Kali (`105`) is scoped the same
way to the operator's host, but deliberately carries no gateway fallback — it is
not an administrative relay for other hosts, and adding one would make the
restriction decorative rather than enforced.

**Provenance:** re-exported from live host state on 2026-09-27 via
`ssh pve "cat /etc/pve/firewall/<n>.fw"`. Supersedes the 2026-08-30 snapshot,
which predated the jump-host restriction described above and the addition of
`105`/`106`. An earlier snapshot committed 2026-08-19 predated the HMI→historian
conduit (`102`) and the gateway-SSH rules (`101`, `102`), and omitted `100`
entirely. Regenerate this set whenever the deployed policy changes.
