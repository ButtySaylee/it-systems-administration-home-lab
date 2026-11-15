# Lab Progress and Session Handover

> **For my AI instructor:** Paste this file at the start of each new session. Continue from **Current position**. Verify anything marked *unverified* before relying on it.

**Last updated:** 2026-11-15

## Programme

A personal training programme in nine phases, aimed at job-ready infrastructure skills:

0. Lab foundations
1. Windows Server and Active Directory
2. Networking (CCNA depth)
3. Linux and scripting
4. IT operations
5. Security
6. Cloud and hybrid identity
7. Infrastructure as code
8. Capstones and career

Teaching method: one stage at a time, ticket-style exercises, evidence required before moving on, and a write-up after every lab.

## Current position

- **Phase 1:** Windows Server and Active Directory
- **Completed:** Lab 0.1 (physical network), Lab 0.2 (VMware networking), Design Decision 001, Lab 1.1 (DC01 preparation), CHG-001 to CHG-005
- **Next:**
  1. Take snapshot `02 - DC01 prepared - before AD DS`.
  2. Lab 1.2: install AD DS and promote DC01, creating the forest `ad.buttytechnologies.test` with the NetBIOS name `BUTTYTECH` (not the default `AD`).

## Verified environment facts

| Item | Value |
|---|---|
| Host PC | ButtyTheGreat · Windows 11 Pro · i5 11th Gen · 32 GB · VMware Workstation Pro · Wi-Fi 192.168.1.103 (DHCP) · DNS: router · profile Private |
| Client PC | SUCCESS · Windows 11 Pro · 4 GB · Wi-Fi 192.168.1.102 (DHCP) · profile Private |
| Router | Huawei B612 4G LTE (SIM) · 192.168.1.1 · DHCP .100–.200 · 1-day lease |
| Safe static ranges | 192.168.1.2–99 and .201–254 |
| VMware | VMnet0 Bridged → Intel Wi-Fi AX201 · VMnet1 Host-only 192.168.159.0/24 · VMnet8 NAT 192.168.160.0/24 |
| DC01 | Windows Server 2025 Std Eval (Desktop Experience) · 6 GB · 2 vCPU · 80 GB · Bridged · Ethernet0 · **192.168.1.10/24 static** · GW 192.168.1.1 · DNS 192.168.1.1 (temporary) · Greenwich Standard Time · profile Private · ICMP allowed (Domain and Private) |
| DC01 snapshots | `01 - Baseline - clean install before configuration` |
| DC01 licence | Evaluation, about 172 days remaining as of 2026-11-15 |
| Repository | github.com/ButtySaylee/it-systems-administration-home-lab · `main` · local Git identity uses the GitHub no-reply email |
| Local repository path | `~/Desktop/IT-Home-Lab/it-systems-administration-home-lab` |

## Decisions made

- Simulated organization: Butty Technologies; domain `ad.buttytechnologies.test`; NetBIOS `BUTTYTECH`
- Design Decision 001: domain on the existing LAN for Phase 1; segmentation with pfSense in Phase 2
- Domain members use DC01 as their **only** DNS server; the host keeps the router as its DNS server
- The router is the only DHCP server on the LAN; no router changes without discussion
- Commit directly to `main` for documentation

## Open items

- [ ] Lab 0.1 screenshots, redacted, including SUCCESS evidence
- [ ] Lab 0.2 and 1.1 screenshots
- [ ] Detach the installation ISO from DC01's CD/DVD drive (housekeeping)
- [ ] Move both PCs to Ethernet before domain join, if cables are available
- [ ] Plan the DC01 evaluation licence before it expires
- [ ] Watch for the DC network-profile quirk after promotion (should become DomainAuthenticated)

## Change log

See [CHANGELOG.md](CHANGELOG.md).
