# Lab 0.3 — Network Migration to a New Router

**Organization:** Butty Technologies (simulated)
**Category:** Network administration · Change management · Active Directory operations
**Status:** Complete
**Related:** [Design Decision 002](network-design.md) · [CHG-008 to CHG-010](../CHANGELOG.md) · Previous network: [Lab 0.1](physical-network-assessment.md)

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
| Unidentified device | — | .115, locally administered (randomised) MAC. Not SUCCESS; investigation open. |

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
| SUCCESS → DC01 | Replies ✓ |

## 7. Problems encountered

**Unexpected shutdown.** `dcdiag` reported event **6008** (*"The previous system shutdown at 3:06:36 PM was unexpected"*) and Kernel-Power event **41**. DC01 had been stopped without a clean Windows shutdown. The DFS Replication log showed event **2214**: *"successfully recovered from an unexpected shutdown… No user action is required."* SYSVOL is healthy.

**Tool quirk.** A misspelled property in `Select-Object` (`LevelDiplayName`) produced an empty column with no error. An unexpectedly empty column means: check the spelling first.

## 8. Lessons learned

- **Assess before changing.** The new network had a different subnet, double NAT, and a DHCP pool covering almost every address.
- **One change ripples:** moving the host to Ethernet silently disconnected the VM, because the bridge was pinned to Wi-Fi.
- **Design trade-offs:** a small, reversible change now (shrinking the pool) beats boxing the lab into nine spare addresses.
- **Look before deleting.** Stale DNS records were expected, but none existed; a blind cleanup would have "fixed" nothing.
- **Shut down domain controllers cleanly.** A hard power-off risks the AD and SYSVOL databases. Here DFSR recovered on its own, but in other cases replication stops and waits for an administrator (event 2213).
- **Document migrations as new labs.** Earlier write-ups stay accurate for the network they describe.
