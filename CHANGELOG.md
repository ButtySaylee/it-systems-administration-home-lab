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
