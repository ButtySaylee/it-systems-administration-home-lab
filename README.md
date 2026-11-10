# IT Systems Administration Home Lab

A practical, hands-on home lab documenting my development of professional IT infrastructure, systems administration, networking, troubleshooting, and cybersecurity skills.

## About This Project

This project simulates the IT environment of a small organization called **Butty Technologies**. It is designed to provide practical experience with the technologies and responsibilities commonly encountered in IT Support, Help Desk, Desktop Support, IT Operations, and Junior Systems Administration roles.

Rather than focusing only on theory, I am building, configuring, testing, troubleshooting, and documenting real systems.

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
| Network | Physical Wi-Fi router and bridged virtual networking |

Sensitive network details and credentials are intentionally excluded from this public repository.

## Planned Lab Architecture

- **DC01** — Windows Server domain controller, DNS, and Active Directory services.
- **CLIENT01** — Windows 11 workstation for employee-side testing.
- **CLIENT02** — Additional Windows client for administration and troubleshooting exercises.
- **Linux Server** — Linux administration and service-management practice.

The proposed lab domain is `ad.buttytechnologies.test`.

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
- PowerShell automation
- Endpoint security and hardening
- Vulnerability assessment and security monitoring
- Virtualization

These are learning objectives; each skill will be marked as completed only after practical implementation and testing.

## Progress

### Completed

- [x] Installed VMware Workstation Pro.
- [x] Prepared the home lab directory structure.
- [x] Downloaded the Windows Server 2025 installation ISO.
- [x] Created the first Windows Server virtual machine.
- [x] Configured bridged networking for the server VM.
- [x] Established network connectivity between the lab server and physical computers.

### In Progress

- [ ] Document the lab environment and network topology.
- [ ] Finalize the server's network configuration.
- [ ] Configure Windows Server roles and features.
- [ ] Install and configure Active Directory Domain Services.
- [ ] Configure DNS and domain services.

## Documentation Structure

Each lab will include, where applicable:

1. Objective and business scenario.
2. Environment and prerequisites.
3. Implementation steps.
4. Commands and configuration details.
5. Screenshots and test evidence.
6. Problems encountered and troubleshooting.
7. Results and lessons learned.

## Project Roadmap

The lab will progressively cover infrastructure setup, Windows Server, Active Directory, DNS, DHCP, Group Policy, Windows client management, file services, remote support, backup, help desk operations, networking, Linux, endpoint security, automation, Microsoft 365, deployment, vulnerability assessment, security monitoring, and a final enterprise simulation.

## Purpose

My goal is to develop practical, demonstrable skills for professional IT infrastructure and systems administration roles through continuous hands-on learning and documented projects.

---

**Project:** IT Systems Administration Home Lab  
**Simulated Organization:** Butty Technologies  
**Repository:** [GitHub Home Lab](https://github.com/ButtySaylee/it-systems-administration-home-lab)
