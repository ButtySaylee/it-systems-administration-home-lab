# Change Log — Butty Technologies IT Lab

All configuration changes to lab and physical systems are recorded here. Each entry includes the reason, the exact change, verification and a rollback method.

---

## CHG-001 — Restrict inbound ICMP to trusted networks

| Field | Detail |
|---|---|
| Date | 2026-11-14 |
| Implemented by | Butty Saylee |
| Systems | ButtyTheGreat, SUCCESS |
| Category | Host firewall / network profile |
| Risk | Low |
| Related lab | [Lab 0.1](01-home-lab-infrastructure/physical-network-assessment.md) |

**Reason:** An undocumented change had enabled the inbound ICMP echo rule on both the Private and Public profiles, and the home network was classified as Public. As a result, the PCs would answer pings on any untrusted network they joined.

**Before**

- NetworkCategory: Public
- `File and Printer Sharing (Echo Request - ICMPv4-In)`: Enabled, profiles Private and Public

**Change** (elevated PowerShell)

```powershell
Set-NetConnectionProfile -InterfaceAlias "<adapter>" -NetworkCategory Private
Get-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)" |
    Where-Object { $_.Enabled -eq 'True' -and $_.Profile -match 'Public' } |
    Set-NetFirewallRule -Profile Private
```

Adapter names: `Wi-Fi 4` (ButtyTheGreat), `Wi-Fi` (SUCCESS).

**After**

- NetworkCategory: Private
- Echo Request rule: Enabled, Private profile only

**Verification:** `Get-NetConnectionProfile`, `Get-NetFirewallRule` and ping tests in both directions passed.

**Rollback**

```powershell
Set-NetConnectionProfile -InterfaceAlias "<adapter>" -NetworkCategory Public
Get-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)" |
    Where-Object Enabled -eq 'True' |
    Set-NetFirewallRule -Profile Private, Public
```

**Status:** Closed — successful.

---

## CHG-002 — Rename server to DC01

| Field | Detail |
|---|---|
| Date | 2026-11-14 |
| System | DC01 (was WIN-7ABKJRGOSUV) |
| Risk | Low (pre-promotion) |
| Prerequisite | Snapshot `01 - Baseline - clean install before configuration` |
| Related lab | [Lab 1.1](02-windows-server/dc01-preparation.md) |

**Reason:** The server needs its permanent name before AD DS promotion; renaming a domain controller afterwards is complex.
**Change:** `Rename-Computer -NewName "DC01" -Restart`
**Verification:** `hostname` → DC01
**Rollback:** Revert the snapshot, or `Rename-Computer -NewName "<old name>" -Restart`
**Status:** Closed — successful.

---

## CHG-003 — Correct DC01 time zone

| Field | Detail |
|---|---|
| Date | 2026-11-14 |
| System | DC01 |
| Risk | Low |

**Reason:** The time zone was set to Pacific Standard Time (UTC−8). The underlying UTC clock was correct, but local timestamps in logs would have been 8 hours off.
**Change:** `Set-TimeZone -Id "Greenwich Standard Time"`
**Verification:** `Get-TimeZone` → (UTC+00:00) Monrovia, Reykjavik; the clock matches the host.
**Rollback:** `Set-TimeZone -Id "Pacific Standard Time"`
**Status:** Closed — successful.

---

## CHG-004 — Static IP for DC01

| Field | Detail |
|---|---|
| Date | 2026-11-15 |
| System | DC01, adapter Ethernet0 |
| Risk | Medium (network change; performed from the VM console to avoid losing access) |
| Design | [Design Decision 001](01-home-lab-infrastructure/network-design.md) |

**Reason:** A domain controller requires a fixed address. The DHCP address had already changed (.106 → .107).
**Before:** DHCP, 192.168.1.107, gateway and DNS 192.168.1.1
**Pre-check:** No ARP response for 192.168.1.10.
**Change:**

```powershell
Set-NetIPInterface -InterfaceAlias "Ethernet0" -Dhcp Disabled
Remove-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
Remove-NetRoute -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.1.10 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.1.1
```

**After:** 192.168.1.10/24 (Preferred), gateway 192.168.1.1, DNS 192.168.1.1 (temporary until AD DS), DHCP disabled
**Verification:** `ipconfig /all`; ping to the gateway; `Resolve-DnsName google.com`
**Rollback:**

```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
Remove-NetRoute -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
Set-NetIPInterface -InterfaceAlias "Ethernet0" -Dhcp Enabled
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ResetServerAddresses
```

**Status:** Closed — successful.

---

## CHG-005 — Allow ICMP echo on DC01 for trusted networks

| Field | Detail |
|---|---|
| Date | 2026-11-15 |
| System | DC01 |
| Risk | Low |

**Reason:** Pings to DC01 were dropped (root cause traced in Lab 1.1). ICMP is needed for routine troubleshooting.
**Before:** Network profile Public; `FPS-ICMP4-ERQ-In` Disabled, profile Any
**Change:**

```powershell
Set-NetConnectionProfile -InterfaceAlias "Ethernet0" -NetworkCategory Private
Set-NetFirewallRule -Name "FPS-ICMP4-ERQ-In" -Profile Domain, Private -Enabled True
```

**After:** Profile Private; rule Enabled, Domain and Private only
**Verification:** Host → DC01, 4/4 replies, TTL=128
**Rollback:**

```powershell
Set-NetFirewallRule -Name "FPS-ICMP4-ERQ-In" -Enabled False -Profile Any
Set-NetConnectionProfile -InterfaceAlias "Ethernet0" -NetworkCategory Public
```

**Status:** Closed — successful.
