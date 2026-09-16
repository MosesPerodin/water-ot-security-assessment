# Assessment Methodology

Project-knowledge reference. This file holds the *reasoning* behind the
assessment approach — the parts not recoverable by scanning the lab or the
repo. Live state lives in `LAB-TOPOLOGY.md`. Artifact status lives in
`PORTFOLIO-STATE.md`. Finding cross-references live in `FINDINGS-REGISTER.md`.

---

## 1. Scope and framing

The portfolio documents a **risk assessment**, not a penetration test. Every
artifact should read as the work of an assessor establishing whether controls
exist and function — not an operator demonstrating exploitation.

- Subject: Shenandoah Valley Water Authority (fictional utility)
- Lab is a *representative environment*, not a claim of a real utility network
- Where the lab diverges from a real plant, the artifact says so explicitly
- Exploitation activity is scoped to what an assessor would legitimately do to
  validate a control (e.g. credential test against a known default), never to
  what a red team would do to establish persistence or move laterally

**Lab construction note:** the author built the environment assessed in this portfolio. The findings documented here reflect defects introduced and subsequently surfaced by the assessment procedure; they are not claims about a pre-existing utility network, and they demonstrate the assessment methodology applied to a known-state environment rather than discovered on an unfamiliar production system.

Anything that cannot be framed as "an assessor verifying a control" does not
belong in the portfolio.

---

## 2. Frameworks in scope

| Framework | Role in the portfolio |
|---|---|
| IEC 62443 | Zone/conduit model, security levels, crosswalk target |
| NIST SP 800-82 | OT-specific control guidance and rationale |
| NIST SP 800-53 | Control catalogue, crosswalk source |
| Purdue Model | Zone framing for segmentation work |
| MITRE ATT&CK for ICS | Technique mapping, case-study structure |
| AWIA §2013 | Risk and resilience assessment driver |
| EPA 817-B-23-001 | Water-sector cybersecurity checklist |
| AWWA assessment framework | Sector self-assessment |
| NIST AI RMF 1.0 | Destination framework (Artifact 9, post-MVP) |

Control identifiers are **verified against primary sources** before they enter a
document. A plausible-looking control ID is a liability, not a detail.

---

## 3. Phase order — strict, non-negotiable

The evidentiary value of the whole segmentation artifact depends on this order.
It cannot be repaired after the fact.

1. **Capture before-evidence** — full test set, all zone pairs, recorded
2. **Service baseline** — what is actually listening, per host
3. **Implement segmentation** — firewall/routing changes
4. **Repeat identical tests** — same commands, same flags, same source hosts
5. **Compare and document delta**

**No firewall or routing change is made until the complete before-set is
captured.** A single early change invalidates the before-state, and there is no
way to reconstruct it honestly.

If a change is required mid-phase (e.g. to install a package), the change is
made, reverted, and *noted in the artifact* — never quietly absorbed.

---

## 4. Evidence rules

- Every claim in an artifact traces to a captured command output
- Evidence is exported and retained outside the VM that produced it
- Command, source host, destination, and flags are all recorded — a result
  without its source host is not evidence of anything
- Re-verification uses the **identical method** as the original test. A
  remediation confirmed by a different method is a different claim.
- Nothing is stated as verified until it has actually been verified. If state
  is unknown, the document says UNVERIFIED and names the check that would
  resolve it.

---

## 5. Two errors this project explicitly guards against

### 5.1 Same-subnet sweeps are not cross-zone evidence

An nmap sweep from a host to other hosts **on its own subnet** is local-zone
asset discovery. The traffic is bridged at L2 and never traverses the FORWARD
chain, so it says nothing whatsoever about inter-zone reachability — before
segmentation or after.

Any "before" claim about cross-zone access requires a scan **sourced from a
host in a different zone**. This is the single easiest way to produce an
artifact that looks rigorous and proves nothing.

### 5.2 Scanning restraint is itself a deliverable

`-T3` timing is a deliberate choice, not a default left unexamined. Aggressive
scan timing can crash legacy PLCs and RTUs; an assessor who scans an OT network
at `-T5` has demonstrated they don't understand the environment.

**Document the restraint and the reason.** In an OT assessment portfolio, the
decision not to do something is as much a methodology artifact as the scan
output. This is a differentiator against IT-background assessors and should be
visible in the write-up, not buried in a command log.

---

## 6. Layer-by-layer validation

Before concluding a service-level failure, walk the stack:

1. **L2** — link state, bridge membership, MAC visibility, ARP resolution
2. **L3** — addressing, routing table, forwarding path, NAT, conntrack
3. **Filtering** — FORWARD/INPUT/POSTROUTING chains, counters, and whether the
   packet is even reaching the chain being blamed
4. **Service** — listener present, bound interface, application-layer auth

Counters matter. A rule with a frozen counter is not filtering the traffic in
question — it is not being reached. Diagnosing "the firewall blocked it" when
the packet never arrived at that chain produces a confidently wrong finding.

**Platform caveat established in this lab:** with `firewall=1` on a VM NIC,
Proxmox inserts an `fwbr/fwln/fwpr` veth layer; with `bridge-nf-call-iptables`
active, `br_netfilter` hooks can cause `nf_nat` to mark flows NAT-complete
before POSTROUTING, so MASQUERADE never fires. Verify the mechanism is present
before attributing any egress failure to it — see LAB-TOPOLOGY.md.

---

## 7. Consequence-based finding framing

Findings are framed by **operational consequence**, not raw CVSS. CVSS may
appear as a secondary reference; it is never the headline.

Every finding answers: *what does this let someone do to the process?*

Framing dimensions:

- **Process uptime** — can this interrupt treatment or conveyance?
- **Setpoint manipulation** — can a control value be altered, and what is the
  physical result?
- **Safety system analog** — does this touch, bypass, or mask a protective
  function?
- **Operator visibility** — can the HMI be made to show something untrue?
- **Recovery** — how would the plant detect and reverse this?

Write the consequence in plant terms an operations manager would recognise.
This is where the 10 years of operations experience is load-bearing, and it is
the part a pure-IT assessor cannot easily reproduce.

Bad: `CVE-XXXX-YYYY, CVSS 9.8, default credentials on OpenPLC.`

Good: `Unauthenticated access to the control runtime permits modification of
process setpoints and forcing of output coils, with no corresponding change
visible at the HMI. In a plant context this is a chemical-dosing and
process-uptime exposure, not solely a data-confidentiality issue.`

---

## 8. Finding lifecycle and identifier convention

**Canonical format is `Finding N`** — matching the existing corpus. Do not
renumber to a new scheme.

Rules:

- **Status-bearing statements always name a discrete finding.** Never
  `Findings 1–4` or `Findings 1 and 2` in a line that asserts status. Range
  and conjunction forms are invisible to a discrete-ID grep and are how
  contradictory statuses survive a remediation sweep.
- Ranges are acceptable **only** in narrative prose carrying no status claim.
- Every finding gets a numeric ID at creation. `(New)` is not an identifier.
- Status vocabulary is fixed: `Open` | `Remediated (date)` | `Accepted risk` |
  `Out of scope`.

**Remediation procedure:**

1. Re-test using the identical method as the original finding
2. Record the re-test output as evidence
3. Update `FINDINGS-REGISTER.md` first — it lists every referencing document
4. Update every referencing document from that list
5. Re-grep for the finding ID before committing; confirm zero contradictions
6. Commit all documents together in a single commit

Step 3 exists because a remediation is not one edit — it is N edits across N
documents, and the register is what makes N knowable rather than remembered.

---

## 9. Document conventions

- Filenames: Title Case, hyphenated — `Artifact-3-Segmentation-Assessment.md`
- Location: `docs/artifacts/`
- **New artifacts match the structure of existing artifacts.** Read a completed
  one first and mirror its heading hierarchy, evidence-block style, and finding
  presentation. Match, don't reinvent.
- Positioning statement, where included, reads: *wastewater operations
  professional, 10 years plant management, worked with Ignition SCADA HMI for
  monitoring and operational insight, pivoting to OT/ICS cybersecurity
  assessment.* Never "SCADA expert" or "SCADA programmer."
- Disputed or contested facts carry the dispute inline (e.g. the Oldsmar 2021
  attribution caveat). Assessments that overstate certainty age badly.
- Cross-references are reconciled across all documents **before** committing.

---

## 10. Standing verification discipline

- `pve-firewall compile` is a CLI preview and does **not** reliably reflect
  kernel-enforced state. Use live `iptables -L -n -v` and `ebtables -L`.
- `qm config` contains no guest IP. Resolve IPs from observed state
  (`ip -4 neigh`) with the raw table retained, never from bridge subnet
  inference.
- `BC:24:11:*` is the Proxmox virtual NIC OUI. nmap reporting "Proxmox Server
  Solutions GmbH" identifies a **VM**, not the hypervisor.
- Two firewall subsystems coexist on PVE 9.x — legacy `pve-firewall`
  (iptables/ebtables) and `proxmox-firewall` (nftables). Confirm which is
  active before reading rules from either.
