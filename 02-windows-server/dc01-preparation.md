# Lab 1.1 — Domain Controller Preparation (DC01)

**Organization:** Butty Technologies (simulated)
**Category:** Windows Server administration · Network configuration
**Status:** Complete
**Related changes:** [CHG-002 to CHG-005](../CHANGELOG.md) · **Design:** [Design Decision 001](../01-home-lab-infrastructure/network-design.md)

---

## 1. Business scenario

Butty Technologies needs its first domain controller. Before Active Directory Domain Services is installed, the server must have a recovery point, a permanent name, a correct clock, a static IP address, and controlled reachability for troubleshooting.

## 2. Learning objectives

- Create a recovery point before making changes.
- Rename a server before promotion, and understand why the order matters.
- Understand time-zone vs UTC and why Kerberos depends on time.
- Confirm an address is free before assigning it.
- Configure a static IPv4 address with PowerShell.
- Diagnose a failed ping layer by layer (ICMP → ARP → firewall).
- Allow ICMP using least exposure.

## 3. Starting state

| Item | Value |
|---|---|
| Hostname | WIN-7ABKJRGOSUV |
| Time zone | Pacific Standard Time (UTC−8), incorrect |
| IPv4 | DHCP, 192.168.1.106, later changed to .107 by the router |
| Network profile | Public |
| Ping rule `FPS-ICMP4-ERQ-In` | Disabled, profile Any |

## 4. Implementation

### 4.1 Baseline snapshot

Shut down the VM, then **VM → Snapshot → Take Snapshot**, named `01 - Baseline - clean install before configuration`. Taking it while powered off gives a smaller, cleaner snapshot. Verified in Snapshot Manager.

> A snapshot isn't a backup: it depends on the original virtual disk and lives on the same physical drive.

### 4.2 Rename (CHG-002)

```powershell
Rename-Computer -NewName "DC01" -Restart
```

Renaming is done **before** promotion, because renaming a domain controller afterwards is a complex, multi-step job.

### 4.3 Time zone (CHG-003)

The clock's underlying UTC time was correct: 08:40 PST is 16:40 UTC, which matched the host. Only the display zone was wrong.

```powershell
Set-TimeZone -Id "Greenwich Standard Time"
```

### 4.4 Address availability check

```powershell
ping 192.168.1.10
arp -a | findstr /C:"192.168.1.10 "
```

The ping triggered an ARP request, and no device answered for 192.168.1.10, so the address was free.

### 4.5 Static IP (CHG-004)

```powershell
Set-NetIPInterface -InterfaceAlias "Ethernet0" -Dhcp Disabled
Remove-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
Remove-NetRoute -InterfaceAlias "Ethernet0" -AddressFamily IPv4 -Confirm:$false
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.1.10 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.1.1
```

DNS points to the router **temporarily**. It will point to DC01 itself after AD DS and DNS are installed.

### 4.6 ICMP for trusted networks (CHG-005)

```powershell
Set-NetConnectionProfile -InterfaceAlias "Ethernet0" -NetworkCategory Private
Set-NetFirewallRule -Name "FPS-ICMP4-ERQ-In" -Profile Domain, Private -Enabled True
```

The rule is targeted by its unique `-Name` rather than its display name.

## 5. Testing and verification

| Test | Result |
|---|---|
| `hostname` | DC01 ✓ |
| `Get-TimeZone` | Greenwich Standard Time (UTC+00:00) ✓ |
| `ipconfig /all` | DHCP: No · 192.168.1.10 (Preferred) · /24 · GW and DNS 192.168.1.1 ✓ |
| DC01 → router | 4/4 replies ✓ |
| `Resolve-DnsName google.com` | A and AAAA records returned ✓ |
| Profile and rule | Private · `FPS-ICMP4-ERQ-In` True, Domain and Private ✓ |
| ButtyTheGreat → DC01 | 4/4 replies, <1 ms, TTL=128 ✓ |

## 6. Problems encountered and troubleshooting

### 6.1 DC01's DHCP address changed (.106 → .107)

After restarts, the router assigned a different address. This demonstrated why a domain controller requires a static IP.

### 6.2 Ping from the host to DC01 timed out

| Step | Evidence | Conclusion |
|---|---|---|
| 1 | `ping` → *Request timed out* (not *Destination host unreachable*) | Something likely received the ping but didn't reply |
| 2 | `arp -a` immediately after ping → `192.168.1.10 → 00-0c-29-3a-0e-80` | DC01 answered ARP. The MAC matched DC01's adapter, and `00-0C-29` is VMware's prefix. Layer 2 is fine. |
| 3 | `Get-NetFirewallRule` → `FPS-ICMP4-ERQ-In` Enabled: False | **Root cause:** the server firewall drops ICMP echo requests by default |

Resolved by CHG-005.

### 6.3 Tool quirks encountered

- In Windows PowerShell 5.1, `Test-Connection` reported *"A non-recoverable error occurred during a database lookup"* for a simple no-reply. Cross-checking with `ping` clarified it.
- In `findstr`, spaces separate search terms. Use `/C:"exact phrase"` for literal matches.
- ARP entries expire quickly, so the check must run immediately after the ping.
- `Set-TimeZone` failed with `TimeZoneNotFound` because of a typo. `Get-TimeZone -ListAvailable` lists valid IDs.

## 7. Lessons learned

- **Create a recovery point before changing anything.**
- **Do the irreversible-later tasks first:** rename before promotion.
- **Correct timestamps matter** for logs and investigations, even when Kerberos would tolerate a wrong display zone.
- **Troubleshoot layer by layer.** ARP proved the network path worked and isolated the fault to the host firewall.
- **When a tool's output surprises you, check how the tool works.**
- **Ping latency reveals the traffic path:** the host reaches its own bridged VM in under 1 ms, while PC-to-PC traffic over Wi-Fi took 40 ms or more.
