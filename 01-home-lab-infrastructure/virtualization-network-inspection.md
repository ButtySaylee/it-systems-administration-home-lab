# Lab 0.2 — Virtualization Network Inspection

**Organization:** Butty Technologies (simulated)
**Category:** Virtualization · Network administration
**Status:** Complete
**Related:** [Lab 0.1](physical-network-assessment.md) · [Network design](network-design.md)

---

## 1. Business scenario

Earlier project notes disagreed on how the server VM connects to the network: one said NAT, another said Bridged. Before any domain services are designed, the IT team must establish the facts by inspecting the hypervisor and the VM. **Inspection only; no changes.**

## 2. Learning objectives

- Understand VMware's three default virtual network types.
- Read the Virtual Network Editor configuration, including hidden networks.
- Check a VM's adapter configuration and confirm it from inside the guest.
- Resolve conflicting documentation using evidence.

## 3. Lab environment

| Component | Detail |
|---|---|
| Host | ButtyTheGreat · Windows 11 Pro · VMware Workstation Pro |
| Host uplink | Intel Wi-Fi 6 AX201 → router (192.168.1.0/24) |
| VM | DC01-Windows-Server · Windows Server 2025 Standard Evaluation (Desktop Experience) · 6 GB RAM · 2 vCPU · 80 GB NVMe |

## 4. Findings

### 4.1 VMware virtual networks

| VMnet | Type | Attached to / subnet | VMware DHCP |
|---|---|---|---|
| VMnet0 | Bridged | Intel Wi-Fi 6 AX201 (pinned, not Automatic) | — (uses the physical LAN's DHCP) |
| VMnet1 | Host-only | 192.168.159.0/24 | Enabled |
| VMnet8 | NAT | 192.168.160.0/24 | Enabled |

- VMnet0 appears in the Virtual Network Editor only after selecting **Change Settings**, which runs the editor with administrator rights.
- Neither VMnet1 nor VMnet8 overlaps the physical LAN.
- VMware's DHCP service only serves its own virtual networks and doesn't conflict with the router. It must be disabled on any VMnet where a Windows DHCP server is deployed later.
- On VMnet8 (NAT), VMware normally uses `.2` as the VMs' gateway and `.1` for the host's virtual adapter.

### 4.2 DC01 VM adapter

| Setting | Value |
|---|---|
| Network connection | Bridged (VMnet0) |
| Connect at power on | Enabled |
| Replicate physical network connection state | Disabled (appropriate for a server) |
| CD/DVD | Installation ISO still attached (housekeeping item) |

### 4.3 DC01 guest state (before configuration)

| Item | Value |
|---|---|
| Hostname | WIN-7ABKJRGOSUV (default) |
| Adapter | Ethernet0 (emulated Intel 82574L) |
| IPv4 | 192.168.1.106 (DHCP, from the router's pool) |
| Gateway and DNS | 192.168.1.1 |
| DHCP enabled | Yes |

## 5. Conclusion

DC01 is **bridged** to the physical LAN through the host's Wi-Fi adapter and receives addresses from the router like any physical device. The repository was correct; the older handover's "NAT" entry was outdated.

## 6. Commands used

```powershell
hostname
ipconfig /all
Get-NetIPConfiguration | Select-Object InterfaceAlias, IPv4Address, IPv4DefaultGateway, DNSServer
Get-NetIPInterface -AddressFamily IPv4 | Where-Object ConnectionState -eq Connected | Select-Object InterfaceAlias, Dhcp
```

## 7. Problems encountered

- **VMnet0 was missing** from the first Virtual Network Editor screenshot. The editor hides bridged networks until it runs with administrator rights.
- **Ambiguous label:** the VM's summary shows "Bridged (Automatic)", even though VMnet0 is pinned to the Wi-Fi adapter. Rather than interpreting the label, the guest's actual address was used as proof.

## 8. Lessons learned

- **Conflicting documentation is resolved by inspection, not by choosing whichever source seems more likely.**
- Close configuration windows with **Cancel** when you only meant to look.
- Guests always see a virtual wired adapter (`Ethernet0`), even when the host uses Wi-Fi.
- Bridging over Wi-Fi works because VMware rewrites the guest's MAC address to the host's for wireless traffic. This can affect MAC-based features such as DHCP reservations.
