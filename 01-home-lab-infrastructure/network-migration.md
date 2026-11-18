# Lab 0.3 — Network Migration to a New Router

**Organization:** Butty Technologies (simulated)
**Category:** Network administration · Change management · Active Directory operations
**Status:** Complete
**Related:** [Design Decision 002](network-design.md) · [CHG-008 to CHG-011](../CHANGELOG.md) · Previous network: [Lab 0.1](physical-network-assessment.md)

---

## 1. Business scenario

The office replaced its router. The new router uses a different subnet, and the domain controller was configured for the old one. IT must assess the new network, redesign the addressing, and move the domain controller without breaking Active Directory.

## 2. Learning objectives

- Assess an unfamiliar network before changing anything.
- Recognise double NAT and its implications.
- Weigh addressing designs on growth, convention, risk and reversibility.
- Change a DHCP scope and observe the DHCP process (DORA).
- Re-point a VMware bridge after the host's uplink changes.
- Re-IP a domain controller and verify DNS registration.
- Analyse an unexpected-shutdown event on a domain controller.

## 3. Assessment of the new network

### 3.1 Topology

```
Internet
   │
Upstream unit (ODU)        192.168.188.1
   │  WAN: 192.168.188.150 (DHCP)
New router                 192.168.5.1   ← double NAT
   │  LAN: 192.168.5.0/24
   ├── ButtyTheGreat (host)   Ethernet (USB adapter)
   │      └── DC01 (VM, bridged)
   └── SUCCESS (client)       Wi-Fi
```

| Item | Value |
|---|---|
| Router | ISP-branded router, MediaTek MT7628 platform |
| LAN | 192.168.5.0/24, gateway 192.168.5.1 |
| WAN | 192.168.188.150 from an upstream outdoor unit at 192.168.188.1 (**double NAT**) |
| DHCP pool (before) | 192.168.5.2 – 192.168.5.244 |
| DNS handed to clients | The router itself (DNS fields empty) |

### 3.2 Hosts and leases

| Device | Connection | Address |
|---|---|---|
| ButtyTheGreat | Ethernet (Realtek USB GbE, `Ethernet 2`); Wi-Fi now disconnected | 192.168.5.55 (DHCP) |
| SUCCESS | Wi-Fi | 192.168.5.166 (DHCP) |
| Two phones | Wi-Fi | .9 and .167 |
| Unidentified device | — | .115, locally administered (randomised) MAC. Confirmed not SUCCESS (different hardware MAC); investigation open. |

Other host adapters noted: a VirtualBox host-only network (192.168.56.0/24) and a TAP-Windows VPN adapter. Neither overlaps lab subnets.

### 3.3 Impact on DC01

| DC01 setting | Value | Problem |
|---|---|---|
| IP / gateway | 192.168.1.10 / 192.168.1.1 | Wrong subnet; gateway no longer exists |
| DNS forwarder | 192.168.1.1 | Internet lookups would fail |
| VMware VMnet0 | Bridged to the Wi-Fi adapter | Wi-Fi disconnected, so DC01 had no network at all |

## 4. Design

See [Design Decision 002](network-design.md): shrink the DHCP pool to .100–.200, place infrastructure statically at .2–.99, and move DC01 to **192.168.5.10**.

## 5. Implementation

### 5.1 CHG-008 — Shrink the DHCP pool

Router → LAN Setting: Address Range changed from `.2–.244` to `.100–.200`. Verified after a page refresh. Then on the host:

```powershell
ipconfig /release "Ethernet 2"
ipconfig /renew "Ethernet 2"     # .55 → 192.168.5.161
```

The release removed the IPv4 address, leaving only the self-assigned IPv6 link-local address. The renew completed a full DHCP exchange (Discover → Offer → Request → Acknowledge) and received an address in the new pool.

### 5.2 CHG-009 — Re-point the VMware bridge

DC01 shut down cleanly. Virtual Network Editor → VMnet0 → **Bridged to: Realtek USB GbE Family Controller**. Then snapshot `03 - AD DS deployed, before network migration`.

### 5.3 CHG-010 — Re-IP DC01

```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
Remove-NetRoute     -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
New-NetIPAddress    -InterfaceAlias "Ethernet0" -IPAddress 192.168.5.10 -PrefixLength 24 -DefaultGateway 192.168.5.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.5.10, 127.0.0.1
Set-DnsServerForwarder -IPAddress 192.168.5.1
ipconfig /registerdns
Restart-Service Netlogon
```

### 5.4 CHG-011 — Set the new network to Private on both PCs

**Finding:** Both PCs showed NetworkCategory **Public** on the new network. Categories are stored per network, so CHG-001 didn't carry over, and the Private-only echo-request rule no longer applied. ButtyTheGreat was also connected twice (Ethernet and Wi-Fi to the same router); Wi-Fi was switched off.

```powershell
Set-NetConnectionProfile -InterfaceAlias "Ethernet 2" -NetworkCategory Private   # ButtyTheGreat
Set-NetConnectionProfile -InterfaceAlias "Wi-Fi"      -NetworkCategory Private   # SUCCESS
```

| Test | Before | After |
|---|---|---|
| ButtyTheGreat → SUCCESS | 100% loss (SUCCESS Public) | 4/4 replies, 57–94 ms |
| SUCCESS → ButtyTheGreat | — | 4/4 replies, 5–24 ms |

The higher latency towards SUCCESS reflects Wi-Fi power saving: an idle wireless client wakes up to receive unsolicited traffic.

## 6. Verification

| Check | Result |
|---|---|
| Address | 192.168.5.10/24, **Preferred** (no duplicate) ✓ |
| DNS client | `{192.168.5.10, 127.0.0.1}` ✓ |
| Forwarder | 192.168.5.1 only ✓ |
| DC01 → router; `Resolve-DnsName google.com` | Replies; resolved ✓ |
| A records, both zones | `@`, `dc01`, `DomainDnsZones`, `ForestDnsZones`, `gc` → **192.168.5.10 only**; no stale entries ✓ |
| SRV `_ldap._tcp.dc._msdcs…` | → dc01, Additional section 192.168.5.10 ✓ |
| AD services | NTDS, KDC, Netlogon, DNS, ADWS Running ✓ |
| Network profile | DomainAuthenticated, IPv4Connectivity Internet ✓ |
| ButtyTheGreat → DC01 | 4/4, <1 ms, TTL=128 ✓ |
| SUCCESS → DC01 | 4/4 replies, 2–7 ms ✓ |
| Network profiles (both PCs) | Private after CHG-011 ✓ |

## 7. Problems encountered

**Unexpected shutdown.** `dcdiag` reported event **6008** (*"The previous system shutdown at 3:06:36 PM was unexpected"*) and Kernel-Power event **41**. DC01 had been stopped without a clean Windows shutdown. The DFS Replication log showed event **2214**: *"successfully recovered from an unexpected shutdown… No user action is required."* SYSVOL is healthy.

**Tool quirk.** A misspelled property in `Select-Object` (`LevelDiplayName`) produced an empty column with no error. An unexpectedly empty column means: check the spelling first.

## 8. Lessons learned

- **Assess before changing.** The new network had a different subnet, double NAT, and a DHCP pool covering almost every address.
- **One change ripples:** moving the host to Ethernet silently disconnected the VM, because the bridge was pinned to Wi-Fi.
- **Design trade-offs:** a small, reversible change now (shrinking the pool) beats boxing the lab into nine spare addresses.
- **Look before deleting.** Stale DNS records were expected, but none existed; a blind cleanup would have "fixed" nothing.
- **Shut down domain controllers cleanly.** A hard power-off risks the AD and SYSVOL databases. Here DFSR recovered on its own, but in other cases replication stops and waits for an administrator (event 2213).
- **Settings tied to a network don't move with the machine.** Network categories, and the firewall rules that depend on them, must be re-checked after a network change.
- **A self-ping proves nothing.** Pinging your own address never touches the firewall; the test must start on another machine.
- **Document migrations as new labs.** Earlier write-ups stay accurate for the network they describe.

## 9. Evidence

Screenshots are stored in [`screenshots/lab-0.3/`](screenshots/lab-0.3/). MAC addresses, DUIDs, router identifiers and network names are redacted.

**Figure 1: New router device information (UUID and MAC redacted)**

![New router device information (UUID and MAC redacted)](screenshots/lab-0.3/01-router-device-info.png)

**Figure 2: WAN address from 192.168.188.0/24: double NAT**

![WAN address from 192.168.188.0/24: double NAT](screenshots/lab-0.3/02-router-wan-double-nat.png)

**Figure 3: Internet type: wired DHCP from the upstream unit**

![Internet type: wired DHCP from the upstream unit](screenshots/lab-0.3/03-router-internet-type.png)

**Figure 4: Before CHG-008: DHCP pool .2–.244**

![Before CHG-008: DHCP pool .2–.244](screenshots/lab-0.3/04-router-dhcp-before.png)

**Figure 5: DHCP client list (MAC addresses redacted)**

![DHCP client list (MAC addresses redacted)](screenshots/lab-0.3/05-router-dhcp-clients.png)

**Figure 6: CHG-008: DHCP pool .100–.200**

![CHG-008: DHCP pool .100–.200](screenshots/lab-0.3/06-router-dhcp-after.png)

**Figure 7: CHG-009: VMnet0 bridged to the Realtek USB Ethernet adapter**

![CHG-009: VMnet0 bridged to the Realtek USB Ethernet adapter](screenshots/lab-0.3/07-vnet-editor-realtek.png)

**Figure 8: Recovery point before the re-IP**

![Recovery point before the re-IP](screenshots/lab-0.3/08-snapshot-03.png)

**Figure 9: CHG-010: new address 192.168.5.10**

![CHG-010: new address 192.168.5.10](screenshots/lab-0.3/09-dc01-new-ip.png)

**Figure 10: DNS client, forwarder and address state Preferred**

![DNS client, forwarder and address state Preferred](screenshots/lab-0.3/10-dns-client-forwarder-preferred.png)

**Figure 11: DNS client {192.168.5.10, 127.0.0.1}**

![DNS client {192.168.5.10, 127.0.0.1}](screenshots/lab-0.3/11-dns-client-verify.png)

**Figure 12: Gateway reachable; internet names resolve**

![Gateway reachable; internet names resolve](screenshots/lab-0.3/12-ping-gateway-resolve.png)

**Figure 13: DNS re-registration and Netlogon restart**

![DNS re-registration and Netlogon restart](screenshots/lab-0.3/13-registerdns-netlogon.png)

**Figure 14: DNS zones (AD-integrated)**

![DNS zones (AD-integrated)](screenshots/lab-0.3/14-dns-zones.png)

**Figure 15: A records in both zones point only to 192.168.5.10**

![A records in both zones point only to 192.168.5.10](screenshots/lab-0.3/15-dns-a-records.png)

**Figure 16: DC01 network profile DomainAuthenticated, IPv4 Internet**

![DC01 network profile DomainAuthenticated, IPv4 Internet](screenshots/lab-0.3/16-network-profile.png)

**Figure 17: SRV record and AD services after migration**

![SRV record and AD services after migration](screenshots/lab-0.3/17-srv-services.png)

**Figure 18: dcdiag: unexpected-shutdown events 6008 and 41**

![dcdiag: unexpected-shutdown events 6008 and 41](screenshots/lab-0.3/18-dcdiag-dirty-shutdown.png)

**Figure 19: Event 2214: DFS Replication recovered automatically**

![Event 2214: DFS Replication recovered automatically](screenshots/lab-0.3/19-dfsr-2214.png)

**Figure 20: Recovery point after migration**

![Recovery point after migration](screenshots/lab-0.3/20-snapshot-04.png)

**Figure 21: ButtyTheGreat ipconfig: virtual adapters (VirtualBox, TAP)**

![ButtyTheGreat ipconfig: virtual adapters (VirtualBox, TAP)](screenshots/lab-0.3/21a-pc1-ipconfig-header.png)

**Figure 22: ButtyTheGreat on Ethernet 192.168.5.161; Wi-Fi disconnected**

![ButtyTheGreat on Ethernet 192.168.5.161; Wi-Fi disconnected](screenshots/lab-0.3/21b-pc1-ipconfig-ethernet-wifi.png)

**Figure 23: ButtyTheGreat → DC01**

![ButtyTheGreat → DC01](screenshots/lab-0.3/22-ping-pc1-to-dc01.png)

**Figure 24: SUCCESS ipconfig header**

![SUCCESS ipconfig header](screenshots/lab-0.3/23a-pc2-ipconfig-header.png)

**Figure 25: SUCCESS on Wi-Fi 192.168.5.166, DNS still the router**

![SUCCESS on Wi-Fi 192.168.5.166, DNS still the router](screenshots/lab-0.3/23b-pc2-ipconfig-wifi.png)

**Figure 26: SUCCESS → DC01**

![SUCCESS → DC01](screenshots/lab-0.3/24-ping-pc2-to-dc01.png)

**Figure 27: Finding: ButtyTheGreat Public on two adapters**

![Finding: ButtyTheGreat Public on two adapters](screenshots/lab-0.3/25-pc1-profile-public-before-chg011.png)

**Figure 28: Wi-Fi switched off; Ethernet still Public**

![Wi-Fi switched off; Ethernet still Public](screenshots/lab-0.3/26-pc1-profile-wifi-off-public.png)

**Figure 29: Finding: SUCCESS Public**

![Finding: SUCCESS Public](screenshots/lab-0.3/27-pc2-profile-public-before-chg011.png)

**Figure 30: Before CHG-011: ButtyTheGreat → SUCCESS times out**

![Before CHG-011: ButtyTheGreat → SUCCESS times out](screenshots/lab-0.3/28-before-chg011-ping-pc1-to-pc2.png)

**Figure 31: CHG-011 on ButtyTheGreat: Private**

![CHG-011 on ButtyTheGreat: Private](screenshots/lab-0.3/29-chg011-profile-pc1.png)

**Figure 32: CHG-011 on SUCCESS: Private**

![CHG-011 on SUCCESS: Private](screenshots/lab-0.3/30-chg011-profile-pc2.png)

**Figure 33: After CHG-011: ButtyTheGreat → SUCCESS replies**

![After CHG-011: ButtyTheGreat → SUCCESS replies](screenshots/lab-0.3/31a-after-chg011-ping-pc1-to-pc2.png)

**Figure 34: After CHG-011: SUCCESS → ButtyTheGreat replies**

![After CHG-011: SUCCESS → ButtyTheGreat replies](screenshots/lab-0.3/31b-after-chg011-ping-pc2-to-pc1.png)
