# Lab Topology — Live State

**Generated on:** 2026-08-30
**Source host:** pve5820 (192.168.1.250)
**Collection method:** read-only; no firewall rule, VM config, or network setting was modified.

> This file is generated. Do not hand-edit it — regenerate it with
> [`tools/gen-lab-topology.sh`](../../tools/gen-lab-topology.sh):
>
> ```bash
> ssh pve "bash -s" < tools/gen-lab-topology.sh > docs/project-knowledge/LAB-TOPOLOGY.md
> ```
>
> (Running it via stdin as above means no script file is written to the Proxmox host.)
> The header above is added by hand when regenerating; everything below this line is script output.

---

## Collection metadata

- Collected (UTC): 2026-08-30 13:44:30
- Collected on host: pve5820
- Collected by: `tools/gen-lab-topology.sh` (read-only)

## VM inventory

Bridge and `firewall=` are read from each VM's first `netN:` line in
`qm config`. Primary IP is resolved by matching that line's MAC address
against the Proxmox host's IPv4 neighbour (ARP) table **on the lab bridges
only** (vmbr10/20/30/40; vmbr0 is excluded so the operator's own LAN devices
are never captured) — so it reflects an address the host has actually
observed, not an assumption from the subnet.
A VM that has not recently sent traffic may therefore show UNVERIFIED even
though it is running and correctly addressed.

Services are not determinable from Proxmox host state (they live inside the
guests) and are intentionally left blank.

| VMID | Name | Primary IP | Bridge | firewall= | Services |
|---|---|---|---|---|---|
| 100 | hmi-scada | 192.168.20.100 | vmbr20 | 1 | |
| 101 | plc-controller | 192.168.10.100 | vmbr10 | 1 | |
| 102 | historian-db | 192.168.20.101 | vmbr20 | 1 | |
| 103 | workstation-eng | 192.168.30.100 | vmbr30 | not set (Proxmox default applies) | |
| 104 | monitor-wireshark | 192.168.40.100 | vmbr40 | 1 | |
| 105 | kali-attack | 192.168.30.50 | vmbr30 | not set (Proxmox default applies) | |

### IPv4 neighbour table, lab bridges only (source for the Primary IP column)

```
192.168.10.100 dev vmbr10 lladdr bc:24:11:a7:63:e0 STALE
192.168.20.100 dev vmbr20 lladdr bc:24:11:f3:89:a8 STALE
192.168.20.101 dev vmbr20 lladdr bc:24:11:d8:2d:2e STALE
192.168.30.50 dev vmbr30 lladdr bc:24:11:66:a4:e7 STALE
192.168.30.100 dev vmbr30 lladdr bc:24:11:14:cd:d8 STALE
192.168.40.100 dev vmbr40 lladdr bc:24:11:6e:bc:a6 STALE
```


## Platform

### pveversion -v

```
proxmox-ve: 9.1.0 (running kernel: 6.17.2-1-pve)
pve-manager: 9.1.1 (running version: 9.1.1/42db4a6cf33dac83)
proxmox-kernel-helper: 9.0.4
proxmox-kernel-6.17.2-1-pve-signed: 6.17.2-1
proxmox-kernel-6.17: 6.17.2-1
ceph-fuse: 19.2.3-pve2
corosync: 3.1.9-pve2
criu: 4.1.1-1
frr-pythontools: 10.3.1-1+pve4
ifupdown2: 3.3.0-1+pmx11
intel-microcode: 3.20250812.1~deb13u1
ksm-control-daemon: 1.5-1
libjs-extjs: 7.0.0-5
libproxmox-acme-perl: 1.7.0
libproxmox-backup-qemu0: 2.0.1
libproxmox-rs-perl: 0.4.1
libpve-access-control: 9.0.4
libpve-apiclient-perl: 3.4.2
libpve-cluster-api-perl: 9.0.7
libpve-cluster-perl: 9.0.7
libpve-common-perl: 9.0.15
libpve-guest-common-perl: 6.0.2
libpve-http-server-perl: 6.0.5
libpve-network-perl: 1.2.3
libpve-rs-perl: 0.11.3
libpve-storage-perl: 9.0.18
libspice-server1: 0.15.2-1+b1
lvm2: 2.03.31-2+pmx1
lxc-pve: 6.0.5-3
lxcfs: 6.0.4-pve1
novnc-pve: 1.6.0-3
proxmox-backup-client: 4.0.20-1
proxmox-backup-file-restore: 4.0.20-1
proxmox-backup-restore-image: 1.0.0
proxmox-firewall: 1.2.1
proxmox-kernel-helper: 9.0.4
proxmox-mail-forward: 1.0.2
proxmox-mini-journalreader: 1.6
proxmox-offline-mirror-helper: 0.7.3
proxmox-widget-toolkit: 5.1.2
pve-cluster: 9.0.7
pve-container: 6.0.18
pve-docs: 9.1.0
pve-edk2-firmware: 4.2025.05-2
pve-esxi-import-tools: 1.0.1
pve-firewall: 6.0.4
pve-firmware: 3.17-2
pve-ha-manager: 5.0.8
pve-i18n: 3.6.2
pve-qemu-kvm: 10.1.2-3
pve-xtermjs: 5.5.0-3
qemu-server: 9.0.30
smartmontools: 7.4-pve1
spiceterm: 3.4.1
swtpm: 0.8.0+pve3
vncterm: 1.9.1
zfsutils-linux: 2.3.4-pve1
```

### qm list

```
      VMID NAME                 STATUS     MEM(MB)    BOOTDISK(GB) PID       
       100 hmi-scada            running    2048               0.00 1695      
       101 plc-controller       running    2048               0.00 1856      
       102 historian-db         running    2048               0.00 1939      
       103 workstation-eng      running    2048               0.00 2021      
       104 monitor-wireshark    running    2048               0.00 2086      
       105 kali-attack          running    4096               0.00 2168      
```

### qm config (per VM)

#### VMID 100

```
boot: c
cores: 8
ide2: local:iso/ubuntu-24.04.4-live-server-amd64.iso,media=cdrom,size=3325654K
memory: 2048
meta: creation-qemu=10.1.2,ctime=1784924802
name: hmi-scada
net0: virtio=BC:24:11:F3:89:A8,bridge=vmbr20,firewall=1
smbios1: uuid=9a5b07e4-01af-4f95-b7a4-aee6e9ee80e1
sockets: 1
virtio0: local-lvm:vm-100-disk-0,size=20G
vmgenid: 0ed487b1-b735-4847-a8df-28cdb7d2bea7
```

#### VMID 101

```
boot: c
cores: 4
ide2: local:iso/ubuntu-24.04.4-live-server-amd64.iso,media=cdrom,size=3325654K
memory: 2048
meta: creation-qemu=10.1.2,ctime=1784925820
name: plc-controller
net0: virtio=BC:24:11:A7:63:E0,bridge=vmbr10,firewall=1
smbios1: uuid=8e4f9617-bf31-4bbf-a4da-fbe3c1c0fbf2
sockets: 1
virtio0: local-lvm:vm-101-disk-0,size=8G
vmgenid: 46e8cbf9-25ce-483e-9357-b70b942a6eb6
```

#### VMID 102

```
boot: c
cores: 4
ide2: local:iso/ubuntu-24.04.4-live-server-amd64.iso,media=cdrom,size=3325654K
memory: 2048
meta: creation-qemu=10.1.2,ctime=1784927124
name: historian-db
net0: virtio=BC:24:11:D8:2D:2E,bridge=vmbr20,firewall=1
smbios1: uuid=c7774690-e1a0-4db8-be2e-d41c3662ed02
sockets: 1
virtio0: local-lvm:vm-102-disk-0,size=10G
vmgenid: b7977e03-2eaa-4b50-9270-2c00ace9c008
```

#### VMID 103

```
boot: c
cores: 4
ide2: local:iso/ubuntu-24.04.4-live-server-amd64.iso,media=cdrom,size=3325654K
memory: 2048
meta: creation-qemu=10.1.2,ctime=1784927614
name: workstation-eng
net0: virtio=BC:24:11:14:CD:D8,bridge=vmbr30
smbios1: uuid=b2556b87-a75e-42da-9606-913bb6177f10
sockets: 1
virtio0: local-lvm:vm-103-disk-0,size=10G
vmgenid: 3ccde89e-465b-447b-91ba-707387e81a85
```

#### VMID 104

```
boot: c
cores: 4
ide2: local:iso/ubuntu-24.04.4-live-server-amd64.iso,media=cdrom,size=3325654K
memory: 2048
meta: creation-qemu=10.1.2,ctime=1784928039
name: monitor-wireshark
net0: virtio=BC:24:11:6E:BC:A6,bridge=vmbr40,firewall=1
smbios1: uuid=be3e4f3d-87fb-48a7-bc9a-a6f756b2cf99
sockets: 1
virtio0: local-lvm:vm-104-disk-0,size=10G
vmgenid: eefb9049-071b-4bb7-be15-510d1ac3ef23
```

#### VMID 105

```
boot: c
bootdisk: virtio0
cores: 2
ide2: local:iso/kali-linux-2025.2-installer-amd64.iso,media=cdrom,size=4373964K
memory: 4096
meta: creation-qemu=10.1.2,ctime=1787005978
name: kali-attack
net0: virtio=BC:24:11:66:A4:E7,bridge=vmbr30
ostype: l26
sata0: local-lvm:vm-105-disk-0,size=20G
smbios1: uuid=aad5b5e5-f748-4d66-8358-26ed95049673
vmgenid: db990769-d9fe-42a5-ba6e-87d5b2b89f85
```

## Networking

### /etc/network/interfaces

```
auto lo
iface lo inet loopback

auto nic0
iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
    address 192.168.1.250/24
    gateway 192.168.1.1
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0

auto vmbr10
iface vmbr10 inet static
    address 192.168.10.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

auto vmbr20
iface vmbr20 inet static
    address 192.168.20.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

auto vmbr30
iface vmbr30 inet static
    address 192.168.30.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

auto vmbr40
iface vmbr40 inet static
    address 192.168.40.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
```

### ip -br addr (vmbr0 IPv6 link-local redacted — see br_addr_redacted)

```
lo               UNKNOWN        127.0.0.1/8 ::1/128 
nic0             UP             
vmbr0            UP             192.168.1.250/24 fe80::[REDACTED-physical-NIC-MAC] 
vmbr10           UP             192.168.10.1/24 fe80::b881:41ff:fe44:e336/64 
vmbr20           UP             192.168.20.1/24 fe80::6079:43ff:fe90:be03/64 
vmbr30           UP             192.168.30.1/24 fe80::dc59:f2ff:fed3:bdab/64 
vmbr40           UP             192.168.40.1/24 fe80::d8bc:56ff:feaf:398b/64 
tap100i0         UNKNOWN        
fwbr100i0        UP             
fwpr100p0@fwln100i0 UP             
fwln100i0@fwpr100p0 UP             
tap101i0         UNKNOWN        
fwbr101i0        UP             
fwpr101p0@fwln101i0 UP             
fwln101i0@fwpr101p0 UP             
tap102i0         UNKNOWN        
fwbr102i0        UP             
fwpr102p0@fwln102i0 UP             
fwln102i0@fwpr102p0 UP             
tap103i0         UNKNOWN        
tap105i0         UNKNOWN        
tap104i0         UNKNOWN        
fwbr104i0        UP             
fwpr104p0@fwln104i0 UP             
fwln104i0@fwpr104p0 UP             
```

### ip route

```
default via 192.168.1.1 dev vmbr0 proto kernel onlink 
192.168.1.0/24 dev vmbr0 proto kernel scope link src 192.168.1.250 
192.168.10.0/24 dev vmbr10 proto kernel scope link src 192.168.10.1 
192.168.20.0/24 dev vmbr20 proto kernel scope link src 192.168.20.1 
192.168.30.0/24 dev vmbr30 proto kernel scope link src 192.168.30.1 
192.168.40.0/24 dev vmbr40 proto kernel scope link src 192.168.40.1 
```

## Packet filtering

### iptables -t nat -L POSTROUTING -n -v

```
Chain POSTROUTING (policy ACCEPT 1433K packets, 91M bytes)
 pkts bytes target     prot opt in     out     source               destination         
  970 59628 MASQUERADE  all  --  *      vmbr0   192.168.40.0/24      0.0.0.0/0           
```

**Note (2026-09-28):** Field's (`192.168.10.0/24`) and Supervisory's (`192.168.20.0/24`) MASQUERADE rules were removed as dead/unused configuration — neither ever carried real traffic (0 packets, confirmed at multiple points across this project). Only Monitoring's rule remains, retained intentionally despite being confirmed non-functional: Finding 3.2 (Artifact 3, Appendix A.1) traces why a MASQUERADE rule with a non-zero packet counter is not evidence of working egress — outbound packets leave correctly translated, but replies are dropped by the guest's own firewall before delivery.

### iptables -L FORWARD -n -v

```
Chain FORWARD (policy ACCEPT 1533K packets, 101M bytes)
 pkts bytes target     prot opt in     out     source               destination         
1561K  112M PVEFW-FORWARD  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
```

### iptables -L INPUT -n -v

```
Chain INPUT (policy ACCEPT 2635 packets, 96380 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 224K   35M PVEFW-INPUT  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
```

### ebtables -L

```
Bridge table: filter

Bridge chain: INPUT, entries: 0, policy: ACCEPT

Bridge chain: FORWARD, entries: 1, policy: ACCEPT
-j PVEFW-FORWARD

Bridge chain: OUTPUT, entries: 0, policy: ACCEPT

Bridge chain: PVEFW-FORWARD, entries: 3, policy: ACCEPT
-p IPv4 -j ACCEPT 
-p IPv6 -j ACCEPT 
-o fwln+ -j PVEFW-FWBR-OUT

Bridge chain: PVEFW-FWBR-OUT, entries: 4, policy: ACCEPT
-i tap100i0 -j tap100i0-OUT
-i tap101i0 -j tap101i0-OUT
-i tap102i0 -j tap102i0-OUT
-i tap104i0 -j tap104i0-OUT

Bridge chain: tap100i0-OUT, entries: 2, policy: ACCEPT
-s ! bc:24:11:f3:89:a8 -j DROP 
-j ACCEPT 

Bridge chain: tap101i0-OUT, entries: 2, policy: ACCEPT
-s ! bc:24:11:a7:63:e0 -j DROP 
-j ACCEPT 

Bridge chain: tap102i0-OUT, entries: 2, policy: ACCEPT
-s ! bc:24:11:d8:2d:2e -j DROP 
-j ACCEPT 

Bridge chain: tap104i0-OUT, entries: 2, policy: ACCEPT
-s ! bc:24:11:6e:bc:a6 -j DROP 
-j ACCEPT 
```

### sysctl net.bridge.bridge-nf-call-iptables

```
net.bridge.bridge-nf-call-iptables = 1
```

## Proxmox firewall configuration

### /etc/pve/firewall/cluster.fw

```
[OPTIONS]
enable: 1

[RULES]
IN ACCEPT -source 192.168.1.0/24 -p tcp -dport 8006
IN ACCEPT -source 192.168.1.0/24 -p tcp -dport 22

# Allow OT network to ping its gateway
IN ACCEPT -source 192.168.10.0/24 -p icmp

# Allow HMI/Historian network to ping its gateway
IN ACCEPT -source 192.168.20.0/24 -p icmp
```

### Per-VM .fw files

#### /etc/pve/firewall/100.fw

```
[OPTIONS]
enable: 1
policy_in: DROP
policy_out: ACCEPT

[RULES]
IN ACCEPT -source 192.168.20.1 -p tcp -dport 22
IN ACCEPT -source 192.168.30.0/24 -p tcp -dport 22

```

#### /etc/pve/firewall/101.fw

```
[OPTIONS]
enable: 1
policy_in: DROP

[RULES]
IN ACCEPT -source 192.168.20.100 -p tcp -dport 502
IN ACCEPT -source 192.168.10.1 -p tcp -dport 22
IN ACCEPT -source 192.168.30.0/24 -p tcp -dport 22
```

#### /etc/pve/firewall/102.fw

```
[OPTIONS]
enable: 1
policy_in: DROP

[RULES]
IN ACCEPT -source 192.168.10.100 -p tcp -dport 5432
IN ACCEPT -source 192.168.20.100 -p tcp -dport 5432
IN ACCEPT -source 192.168.20.1 -p tcp -dport 22
IN ACCEPT -source 192.168.30.0/24 -p tcp -dport 22
```

#### /etc/pve/firewall/104.fw

```
[OPTIONS]
enable: 1
policy_in: DROP

[RULES]
IN ACCEPT -source 192.168.10.0/24 -p tcp -dport 514
IN ACCEPT -source 192.168.20.0/24 -p tcp -dport 514
IN ACCEPT -source 192.168.30.0/24 -p tcp -dport 514
IN ACCEPT -source 192.168.40.0/24 -p tcp -dport 514
IN ACCEPT -source 192.168.30.0/24 -p tcp -dport 22
```

#### /etc/pve/firewall/cluster.fw

```
[OPTIONS]
enable: 1

[RULES]
IN ACCEPT -source 192.168.1.0/24 -p tcp -dport 8006
IN ACCEPT -source 192.168.1.0/24 -p tcp -dport 22

# Allow OT network to ping its gateway
IN ACCEPT -source 192.168.10.0/24 -p icmp

# Allow HMI/Historian network to ping its gateway
IN ACCEPT -source 192.168.20.0/24 -p icmp
```

### systemctl is-active pve-firewall proxmox-firewall

```
active
active
```

