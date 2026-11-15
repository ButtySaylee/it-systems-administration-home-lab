# Network Design — Butty Technologies Lab

Architecture decisions are recorded here with their context, the options considered, and the reasoning behind each choice.

---

## Design Decision 001 — Domain on the existing LAN (Phase 1)

| Field | Detail |
|---|---|
| Date | 2026-11-14 |
| Status | Accepted |
| Revisit | Phase 2 (networking and segmentation) |

### Context

A physical Windows 11 client (SUCCESS) must join the Active Directory domain hosted on the DC01 VM. DC01 is already bridged to the physical LAN (see [Lab 0.2](virtualization-network-inspection.md)).

### Options considered

| Option | Pros | Cons |
|---|---|---|
| **A.** DC01 on the existing LAN, 192.168.1.0/24 | Physical clients reach DC01 directly; simple; no router changes | No segmentation; the lab shares the home network |
| **B.** Isolated lab network behind pfSense | Realistic segmentation and firewall practice | The physical PC can't easily reach isolated VMs; blocks the domain-join exercise |

### Decision

**Option A for Phase 1.** Option B will be built in Phase 2 as a separate segmentation project.

### Design rules

1. **DC01 uses a static IP, 192.168.1.10**, outside the router's DHCP pool (.100–.200).
2. **DC01 is authoritative for `ad.buttytechnologies.test`** and forwards all other DNS queries to the router (192.168.1.1).
3. **Domain members use DC01 as their only DNS server.** The router is never added as a secondary DNS server, because it doesn't hold the domain's SRV records, and querying it causes intermittent authentication and Group Policy failures.
4. **The host (ButtyTheGreat) is not a domain member and keeps the router as its DNS server**, so the host stays usable when the lab is offline.
5. **The router remains the only DHCP server on the LAN.** DC01 never runs DHCP on 192.168.1.0/24.
6. **No router configuration changes.** Client DNS settings are configured on the clients themselves.

### Accepted trade-off

When DC01 is offline, domain members can't resolve names. In production, this is solved with a second domain controller, which is planned for a later lab (SRV02).

### Logical topology

```
                  Router 192.168.1.1
              Gateway · DHCP .100–.200
          ┌──────────────┼──────────────┐
          │              │              │
   ButtyTheGreat       DC01          SUCCESS
   (host, VMware)  (VM, bridged)    (client)
   DHCP             192.168.1.10    DHCP
   DNS: router      AD DS + DNS     DNS: DC01
```
