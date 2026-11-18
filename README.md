# IT Systems Administration Home Lab

A practical, hands-on home lab documenting my development of professional IT infrastructure, systems administration, networking, troubleshooting and cybersecurity skills.

## About This Project

This project simulates the IT environment of a small organization called **Butty Technologies**. It is designed to provide practical experience with the technologies and responsibilities commonly encountered in IT Support, Help Desk, Desktop Support, IT Operations and Junior Systems Administration roles.

Rather than focusing only on theory, I am building, configuring, testing, troubleshooting and documenting real systems. Every configuration change is recorded in a [change log](CHANGELOG.md) with its reason, verification and rollback method. Labs are numbered by phase (0 = foundations, 1 = Windows Server and Active Directory); the change log gives the full chronological history.

## Lab Index

| Lab | Title | Status | Completed |
|---|---|---|---|
| **Phase 0** | **Lab foundations** | | |
| 0.1 | [Physical network assessment](01-home-lab-infrastructure/physical-network-assessment.md) | ✅ Complete | 2026-11-14 |
| 0.2 | [Virtualization network inspection](01-home-lab-infrastructure/virtualization-network-inspection.md) | ✅ Complete | 2026-11-14 |
| 0.3 | [Network migration to a new router](01-home-lab-infrastructure/network-migration.md) | ✅ Complete | 2026-11-18 |
| — | [Network design decisions](01-home-lab-infrastructure/network-design.md) | ✅ Decisions 001–002 | 2026-11-16 |
| **Phase 1** | **Windows Server and Active Directory** | | |
| 1.1 | [Domain controller preparation (DC01)](02-windows-server/dc01-preparation.md) | ✅ Complete | 2026-11-15 |
| 1.2 | [Active Directory forest deployment](03-active-directory/ad-ds-forest-deployment.md) | ✅ Complete | 2026-11-15 |
| 1.3 | Join the physical client to the domain | ⏳ In progress | — |

## Lab Environment

| Component | Configuration |
|---|---|
| Main host | Windows 11 Pro |
| Processor | Intel Core i5, 11th Generation |
| Memory | 32 GB RAM |
| Storage | 1 TB SSD |
| Hypervisor | VMware Workstation Pro |
| Server operating system | Windows Server 2025 Standard Evaluation |
| Physical client | Windows 11 Pro workstation |
| Network | Home router (192.168.5.0/24) behind an upstream unit, with bridged virtual networking |

Full details: [environment](01-home-lab-infrastructure/environment.md) · [network design](01-home-lab-infrastructure/network-design.md). Sensitive network details, hardware identifiers and credentials are intentionally excluded from this public repository.

## Lab Architecture

| System | Role | Status |
|---|---|---|
| **DC01** | Domain controller, DNS, Active Directory · 192.168.5.10 (static) | ✅ Operational |
| **CLIENT01** | Windows 11 VM for employee-side testing | Planned |
| **CLIENT02** | Physical Windows 11 workstation, domain member | Planned |
| **Linux server** | Linux administration and service management | Planned |

Lab domain: `ad.buttytechnologies.test`

## Skills and Technologies

- Windows Server administration
- Active Directory Domain Services
- DNS and DHCP
- Group Policy
- Windows 11 client management
- User and group administration
- File sharing and NTFS permissions
- Networking and troubleshooting
- Remote support and administration
- Backup and disaster recovery
- IT help desk operations and ticket documentation
- Linux system administration
- PowerShell and Bash automation
- Endpoint security and hardening
- Vulnerability assessment and security monitoring
- Virtualization
- Microsoft 365 and cloud identity
- Version control with Git

These are learning objectives; each skill is marked as completed only after practical implementation and testing.

## Progress

### Completed

- [x] Installed VMware Workstation Pro and created the Windows Server 2025 VM.
- [x] Assessed and documented the physical network (Lab 0.1).
- [x] Inspected and documented VMware virtual networking (Lab 0.2).
- [x] Recorded the Phase 1 network design (Design Decision 001).
- [x] Prepared DC01: snapshot, rename, time zone, static IP and controlled ICMP (Lab 1.1).
- [x] Deployed the Active Directory forest `ad.buttytechnologies.test` with integrated DNS (Lab 1.2).
- [x] Migrated the lab to a new router and re-IPed the domain controller without breaking AD (Lab 0.3).
- [x] Verified connectivity between DC01, the host PC and the physical client.

### In Progress

- [ ] Join the physical client to the domain.
- [ ] Create the organizational structure, users and groups.
- [ ] Configure Group Policy.

## Documentation Standard

Each lab includes, where applicable:

1. Objective and business scenario.
2. Environment and prerequisites.
3. Implementation steps.
4. Commands and configuration details.
5. Screenshots and test evidence.
6. Problems encountered and troubleshooting.
7. Results and lessons learned.

## Project Roadmap

The lab will progressively cover infrastructure setup, Windows Server, Active Directory, DNS, DHCP, Group Policy, Windows client management, file services, remote support, backup, help desk operations, networking and segmentation, Linux, endpoint security, automation, Microsoft 365 and cloud identity, deployment, vulnerability assessment, security monitoring, and a final enterprise simulation.

## Purpose

My goal is to develop practical, demonstrable skills for professional IT infrastructure and systems administration roles through continuous hands-on learning and documented projects.

---

**Project:** IT Systems Administration Home Lab  
**Simulated Organization:** Butty Technologies  
**Repository:** [GitHub Home Lab](https://github.com/ButtySaylee/it-systems-administration-home-lab)
