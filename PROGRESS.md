# Lab Progress and Session Handover

> **For my AI instructor:** Paste this file at the start of each new session. Continue from **Current position**. Verify anything marked *unverified* before relying on it.

**Last updated:** 2026-11-18

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

Teaching method: one stage at a time, ticket-style exercises, evidence before moving on, recall practice (the student writes commands before seeing them), and a write-up after every lab.

## Current position

- **Phase 1:** Windows Server and Active Directory
- **Completed:** Labs 0.1, 0.2, 0.3 (network migration), 1.1 and 1.2; Design Decisions 001–002; CHG-001 to CHG-011; 68 redacted evidence screenshots embedded in all lab documents
- **Next:**
  1. Lab 1.3: join SUCCESS (PC2) to the domain. Set its DNS to DC01 only, rename it to CLIENT02 (naming convention), verify the domain login. Consider moving PC2 to Ethernet first.

## Verified environment facts

| Item | Value |
|---|---|
| Host PC | ButtyTheGreat · Windows 11 Pro · i5 11th Gen · 32 GB · VMware Workstation Pro · **Ethernet** via Realtek USB GbE (`Ethernet 2`), DHCP (192.168.5.161 at last check) · DNS: router · Wi-Fi switched off · profile **Private** (CHG-011) |
| Host extras | VirtualBox host-only adapter (192.168.56.1), TAP-Windows VPN adapter |
| Client PC | SUCCESS · Windows 11 Pro · 4 GB · Wi-Fi (Intel AC 9560) · 192.168.5.166 (DHCP) · DNS: router (until Lab 1.3) · profile **Private** (CHG-011) |
| Router | ISP-branded, MT7628 platform · LAN 192.168.5.1/24 · DHCP **.100–.200** · DNS fields empty · WAN 192.168.188.150 via an upstream unit (192.168.188.1), so **double NAT** |
| Address plan | Static infrastructure .2–.99 · DHCP .100–.200 · spare .201–.254 · 192.168.188.0/24 reserved |
| VMware | VMnet0 Bridged → **Realtek USB GbE** · VMnet1 Host-only 192.168.159.0/24 · VMnet8 NAT 192.168.160.0/24 |
| DC01 | Windows Server 2025 Std Eval · **192.168.5.10/24** static · GW 192.168.5.1 · DNS client `{192.168.5.10, 127.0.0.1}` · forwarder 192.168.5.1 (root hints on) · Greenwich Standard Time · DomainAuthenticated · ICMP allowed (Domain and Private) |
| AD | Forest/domain `ad.buttytechnologies.test` · NetBIOS BUTTYTECH · Windows2025Forest/Domain · DC01 is a GC · Default-First-Site-Name · AD-integrated zones; no reverse zone yet |
| DC01 snapshots | 01 Baseline · 02 Prepared before AD DS · 03 AD DS deployed, before migration · 04 Migrated to 192.168.5.0/24, verified |
| DC01 licence | Evaluation; about 172 days remaining as of 2026-11-15 |
| Repository | github.com/ButtySaylee/it-systems-administration-home-lab · `main` · local Git identity uses the GitHub no-reply email |
| Local repository path | `~/Desktop/IT-Home-Lab/it-systems-administration-home-lab` |

## Decisions made

- Simulated organization: Butty Technologies; domain `ad.buttytechnologies.test`; NetBIOS `BUTTYTECH`
- Design Decision 001: domain on the existing LAN in Phase 1; pfSense segmentation in Phase 2
- Design Decision 002: DHCP pool shrunk to .100–.200; DC01 at .10
- Domain members use DC01 as their **only** DNS server; the host keeps the router
- The router is the only DHCP server on the LAN; router changes are recorded changes
- Commit directly to `main` for documentation
- Client naming convention: CLIENT01 (VM), CLIENT02 (physical PC2)

## Open items

- [ ] Re-run `dcdiag /q` after 2026-11-17 16:00; DFSREvent should clear (the dirty-shutdown events then fall outside 24 hours)
- [ ] Always shut DC01 down cleanly (event 6008 on 2026-11-16; DFSR recovered with 2214)
- [ ] Identify the unknown DHCP client (.115, randomised MAC). Confirmed **not** SUCCESS
- [ ] Lab 1.3: after the domain join, SUCCESS's profile becomes DomainAuthenticated, and the echo-request rule is Private-only, so decide how to handle it (Group Policy)
- [ ] Consider deleting snapshots 01 and 02 after Lab 1.3 (practise consolidation)
- [ ] Capture screenshots as we go: one per verification step, `hostname` first
- [ ] Check whether the router's admin password has been changed from the default
- [ ] Inventory: is VirtualBox or the TAP VPN still needed?
- [ ] Keep the USB Ethernet adapter in the same port (the VMnet0 bridge depends on it)
- [ ] Create a reverse lookup zone for 192.168.5.0/24 (DNS lab)
- [ ] Enable Secure Boot on DC01 (hardening, Phase 5)
- [ ] Detach the installation ISO from DC01
- [ ] Plan the evaluation licence before it expires
