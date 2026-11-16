# Network Design — Butty Technologies Lab

Architecture decisions are recorded here with their context, the options considered, and the reasoning behind each choice. Superseded decisions are kept for history.

---

## Design Decision 002 — Addressing on the new network (192.168.5.0/24)

| Field | Detail |
|---|---|
| Date | 2026-11-16 |
| Status | **Accepted.** Supersedes the addressing in Decision 001; its design rules still apply. |
| Related | [Lab 0.3](network-migration.md) |

### Context

The router was replaced. The new LAN is 192.168.5.0/24 behind an upstream unit on 192.168.188.0/24 (double NAT). Its DHCP pool covered .2–.244, leaving only .245–.254 for static addresses.

### Options considered

| Criterion | A. Keep the pool, DC01 at .250 | B. Shrink the pool, DC01 at .10 |
|---|---|---|
| Room to grow | 9 static addresses | About 98 static addresses |
| Convention | Servers scattered at the top of the range | Infrastructure grouped at the low end |
| Risk | No change | One router setting |
| Reversible | — | Yes |

### Decision

**Option B.** A small, reversible change now gives the lab room for SRV02, Linux servers and pfSense.

### Address plan

| Range | Purpose |
|---|---|
| 192.168.5.1 | Router (gateway, DHCP, DNS for non-domain devices) |
| 192.168.5.2 – .99 | Static lab infrastructure; **DC01 = 192.168.5.10** |
| 192.168.5.100 – .200 | Router DHCP pool |
| 192.168.5.201 – .254 | Spare, static-safe |
| 192.168.188.0/24 | **Reserved:** the upstream WAN network; never used in the lab |

### Logical topology

```
                 Upstream unit 192.168.188.1
                            │ (double NAT)
                   Router 192.168.5.1
              Gateway · DHCP .100–.200
          ┌──────────────┼──────────────┐
          │              │              │
   ButtyTheGreat       DC01          SUCCESS
   (host, Ethernet) (VM, bridged)   (client, Wi-Fi)
   DHCP              192.168.5.10   DHCP
   DNS: router       AD DS + DNS    DNS: DC01 (when joined)
```

---

## Design Decision 001 — Domain on the existing LAN (Phase 1)

| Field | Detail |
|---|---|
| Date | 2026-11-14 |
| Status | Accepted. Addressing superseded by Decision 002; the rules below remain in force. |
| Revisit | Phase 2 (networking and segmentation) |

### Context

A physical Windows 11 client must join the domain hosted on the DC01 VM, which is bridged to the physical LAN.

### Options considered

| Option | Pros | Cons |
|---|---|---|
| **A.** DC01 on the existing LAN | Physical clients reach DC01 directly; simple | No segmentation |
| **B.** Isolated lab network behind pfSense | Realistic segmentation | Physical PC can't easily reach isolated VMs |

### Decision

**Option A for Phase 1**; segmentation with pfSense is built in Phase 2.

### Design rules (still in force)

1. **DC01 uses a static IP outside the router's DHCP pool.**
2. **DC01 is authoritative for `ad.buttytechnologies.test`** and forwards other DNS queries to the router.
3. **Domain members use DC01 as their only DNS server.** The router is never a secondary DNS server, because it doesn't hold the domain's SRV records.
4. **The host is not a domain member** and keeps the router as its DNS server, so it works when the lab is offline.
5. **The router is the only DHCP server on the LAN.** DC01 never runs DHCP on it.
6. **Router changes are deliberate, recorded changes.** Client DNS is set on the clients themselves.

### Accepted trade-off

When DC01 is offline, domain members can't resolve names. This is solved later with a second domain controller (SRV02).
