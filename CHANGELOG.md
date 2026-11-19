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

---

## CHG-006 — Promote DC01 to the first domain controller

| Field | Detail |
|---|---|
| Date | 2026-11-15 |
| System | DC01 |
| Risk | High (creates the forest) |
| Prerequisite | Snapshot `02 - DC01 prepared - before AD DS`; `Test-ADDSForestInstallation` → Success |
| Related lab | [Lab 1.2](03-active-directory/ad-ds-forest-deployment.md) |

**Change:** `Install-WindowsFeature AD-Domain-Services -IncludeManagementTools`, then `Install-ADDSForest -DomainName "ad.buttytechnologies.test" -DomainNetbiosName "BUTTYTECH" -InstallDns`
**Verification:** Full Lab 1.2 checks (forest, domain, DC, services, shares, DNS A and SRV records).
**Rollback:** Revert to snapshot 02. Demotion is possible but more complex.
**Status:** Closed — successful.

---

## CHG-007 — DC01 DNS client order

| Field | Detail |
|---|---|
| Date | 2026-11-16 |
| System | DC01 |
| Risk | Low |

**Reason:** Promotion set the DNS client to loopback only. Best practice is the real IP first, loopback last.
**Change:** Applied as `192.168.1.10` only; the loopback entry was omitted.
**Status:** Partially applied — **superseded by CHG-010**, which set `192.168.5.10, 127.0.0.1`.

---

## CHG-008 — Shrink the router's DHCP pool

| Field | Detail |
|---|---|
| Date | 2026-11-16 |
| System | Router (192.168.5.1) |
| Risk | Low |
| Design | [Design Decision 002](01-home-lab-infrastructure/network-design.md) |

**Before:** Pool 192.168.5.2 – .244
**Change:** LAN Setting → Address Range → 192.168.5.100 – .200
**After:** Pool .100 – .200; .2 – .99 reserved for static infrastructure
**Verification:** Setting persisted after refresh; host release/renew gave 192.168.5.161
**Rollback:** Set the range back to .2 – .244
**Status:** Closed — successful.

---

## CHG-009 — Re-point VMware VMnet0 to Ethernet

| Field | Detail |
|---|---|
| Date | 2026-11-16 |
| System | ButtyTheGreat (VMware Workstation) |
| Risk | Low (DC01 powered off) |

**Reason:** The host moved to Ethernet; VMnet0 was pinned to the now-disconnected Wi-Fi adapter, which isolated DC01.
**Before:** VMnet0 bridged to Intel Wi-Fi 6 AX201
**After:** VMnet0 bridged to Realtek USB GbE Family Controller
**Follow-up:** Snapshot `03 - AD DS deployed, before network migration`
**Rollback:** Re-select the Wi-Fi adapter in the Virtual Network Editor
**Status:** Closed — successful.

---

## CHG-010 — Re-IP DC01 to 192.168.5.10

| Field | Detail |
|---|---|
| Date | 2026-11-16 |
| System | DC01 |
| Risk | Medium (DC address change; performed from the VM console) |
| Related lab | [Lab 0.3](01-home-lab-infrastructure/network-migration.md) |

**Before:** 192.168.1.10/24, gateway 192.168.1.1, DNS client 192.168.1.10, forwarder 192.168.1.1
**Change:**

```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
Remove-NetRoute -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.5.10 -PrefixLength 24 -DefaultGateway 192.168.5.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.5.10, 127.0.0.1
Set-DnsServerForwarder -IPAddress 192.168.5.1
ipconfig /registerdns
Restart-Service Netlogon
```

**After:** 192.168.5.10/24 (Preferred), gateway 192.168.5.1, DNS client `{192.168.5.10, 127.0.0.1}`, forwarder 192.168.5.1; all A records point to 192.168.5.10, with no stale entries
**Verification:** Lab 0.3 section 6
**Rollback:** Revert to snapshot 03
**Status:** Closed — successful.

---

## CHG-011 — Set the new network to Private on both PCs

| Field | Detail |
|---|---|
| Date | 2026-11-18 |
| Systems | ButtyTheGreat, SUCCESS |
| Risk | Low |
| Related lab | [Lab 0.3](01-home-lab-infrastructure/network-migration.md) |

**Reason:** Network categories are stored per network. On the new router, both PCs were Public, so the CHG-001 echo-request rule (Private only) no longer applied, and neither PC answered pings.
**Before:** ButtyTheGreat Public (Ethernet 2 and Wi-Fi 4); SUCCESS Public (Wi-Fi). ButtyTheGreat → SUCCESS ping: 100% loss.
**Change:**

```powershell
Set-NetConnectionProfile -InterfaceAlias "Ethernet 2" -NetworkCategory Private   # ButtyTheGreat
Set-NetConnectionProfile -InterfaceAlias "Wi-Fi" -NetworkCategory Private        # SUCCESS
```

ButtyTheGreat's Wi-Fi was also switched off to avoid a duplicate connection to the same network.
**After:** Both Private; pings succeed in both directions.
**Rollback:** The same commands with `-NetworkCategory Public`.
**Status:** Closed — successful.

---

## CHG-012 — Point SUCCESS's DNS to DC01

| Field | Detail |
|---|---|
| Date | 2026-11-18 |
| System | SUCCESS (now CLIENT02), adapter Wi-Fi |
| Risk | Low |
| Related lab | [Lab 1.3](03-active-directory/domain-join-client02.md) |

**Reason:** Domain members must use the domain controller as their only DNS server (Design Rule 3).
**Before:** DNS 192.168.5.1 (router, via DHCP)
**Change:** `Set-DnsClientServerAddress -InterfaceAlias "Wi-Fi" -ServerAddresses 192.168.5.10`
**After:** DNS `{192.168.5.10}`; SRV record and internet names resolve; ports 389, 88 and 445 reachable
**Rollback:** `Set-DnsClientServerAddress -InterfaceAlias "Wi-Fi" -ResetServerAddresses`
**Status:** Closed — successful.

---

## CHG-013 — Rename SUCCESS to CLIENT02 and join the domain

| Field | Detail |
|---|---|
| Date | 2026-11-19 |
| Systems | CLIENT02; DC01 (computer object created) |
| Risk | Medium (physical machine with no snapshot; local account kept as fallback) |
| Related lab | [Lab 1.3](03-active-directory/domain-join-client02.md) |

**Change:** `Add-Computer -DomainName "ad.buttytechnologies.test" -NewName "CLIENT02" -Credential "BUTTYTECH\Administrator"`, then a restart
**Verification:** Domain sign-in; PartOfDomain True; secure channel True; LOGONSERVER \\DC01; profile DomainAuthenticated; time source DC01; AD object and DNS A record present
**Rollback:** `Remove-Computer -UnjoinDomainCredential "BUTTYTECH\Administrator" -WorkgroupName "WORKGROUP" -Restart`, then `Remove-ADComputer CLIENT02` on DC01
**Status:** Closed — successful.
