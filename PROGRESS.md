# Lab Progress and Session Handover

> **For my AI instructor:** Paste this file at the start of each new session. Continue from **Current position**. Verify anything marked *unverified* before relying on it.

**Last updated:** 2026-11-14

## Programme

A personal training programme in nine phases, aimed at job-ready infrastructure skills:

0. Lab foundations
1. Windows Server and Active Directory
2. Networking (CCNA depth)
3. Linux and scripting
4. IT operations (ticketing, backup and recovery, monitoring)
5. Security
6. Cloud and hybrid identity
7. Infrastructure as code
8. Capstones and career

Teaching method: one stage at a time, ticket-style exercises, evidence required before moving on, and a write-up after every lab.

## Current position

- **Phase 0:** Lab foundations
- **Completed:** Lab 0.1 (physical network assessment) and CHG-001
- **Next:** Lab 0.2, inspecting VMware Virtual Network Editor (VMnet0, VMnet1, VMnet8) and the DC01 VM's actual network mode, hostname and IP. Inspect only; no changes.

## Verified environment facts

| Item | Value |
|---|---|
| Primary PC | ButtyTheGreat · Windows 11 Pro · i5 11th Gen · 32 GB · 1 TB SSD · VMware Workstation Pro |
| Secondary PC | SUCCESS · Windows 11 Pro · 4 GB RAM |
| Router | Huawei B612 4G LTE (ISP-provided, SIM-based internet) |
| LAN | 192.168.1.0/24 · gateway, DHCP and DNS at 192.168.1.1 |
| DHCP pool | 192.168.1.100–200 · 1-day lease · no reservations |
| Safe static ranges | 192.168.1.2–99 and 192.168.1.201–254 |
| PC connections | Both on Wi-Fi (ButtyTheGreat: `Wi-Fi 4` = .103; SUCCESS: `Wi-Fi` = .102) |
| Network profile | Private on both PCs (CHG-001) |
| Repository | github.com/ButtySaylee/it-systems-administration-home-lab · branch `main` |
| Local repository path | `~/Desktop/IT-Home-Lab/it-systems-administration-home-lab` |

## Unverified (check in Lab 0.2)

- DC01 VM network mode. The repository says Bridged; an older handover said NAT.
- DC01 Windows hostname. Not yet renamed?
- DC01 current IP address.
- Server-to-PC connectivity. The repository reports success, but it hasn't been re-tested.

## Decisions made

- Simulated organization: **Butty Technologies**
- AD domain: **ad.buttytechnologies.test**. Set the NetBIOS name explicitly (for example `BUTTYTECH`) during promotion, not the default `AD`.
- The router remains the only DHCP server on 192.168.1.0/24. DC01 must never run DHCP on the physical LAN.
- Wired Ethernet is recommended for both PCs before domain join, because of Wi-Fi latency and jitter.
- No changes to the router without discussion first.

## Open items

- [ ] PC2 (SUCCESS) evidence screenshots for Lab 0.1
- [ ] Redact and add Lab 0.1 screenshots
- [ ] Verify Git `user.email` is the GitHub no-reply address
- [ ] Final network design (bridged vs isolated lab network): decide after Lab 0.2

## Change log

See [CHANGELOG.md](CHANGELOG.md).
