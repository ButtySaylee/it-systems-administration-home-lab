# Home Lab Environment Documentation

## 1. Project Overview

This document describes the hardware, software, virtualization platform, and network environment used for the Butty Technologies IT Systems Administration Home Lab.

The environment consists of one primary virtualization host, one physical Windows client, and a Windows Server virtual machine.

## 2. Physical Infrastructure

### 2.1 Primary Computer — Virtualization Host

| Property | Specification |
|---|---|
| Computer name | ButtyTheGreat |
| Operating system | Windows 11 Pro |
| Processor | Intel Core i5, 11th Generation |
| RAM | 32 GB |
| Storage | 1 TB SSD |
| Virtualization | Enabled |
| Hypervisor | VMware Workstation Pro |
| Primary role | Virtualization host and administration workstation |

### 2.2 Secondary Computer — Physical Client

| Property | Specification |
|---|---|
| Computer name | SUCCESS |
| Operating system | Windows 11 Pro |
| RAM | 4 GB |
| Primary role | Physical employee workstation and client testing |

The secondary computer is used as a physical endpoint to simulate an employee workstation in the organization.

## 3. Virtual Infrastructure

### 3.1 Windows Server Virtual Machine

| Property | Configuration |
|---|---|
| Operating system | Windows Server 2025 Standard Evaluation |
| Installation type | Desktop Experience |
| Virtual CPU | 2 vCPUs |
| Memory | 6 GB RAM |
| Virtual disk | 80 GB |
| Hypervisor | VMware Workstation Pro |
| Network mode | Bridged |

**Planned server role:** Active Directory Domain Services, DNS, user and computer administration, Group Policy, and other infrastructure services.

**Server hostname:** To be verified.

**Server IPv4 address:** To be documented after final network configuration.

## 4. Network Environment

### 4.1 Physical Network

The physical computers connect to a Wi-Fi router that provides network connectivity and internet access.

The physical network uses a private IPv4 subnet.

The router is responsible for:
- Providing DHCP address assignments.
- Acting as the default gateway.
- Providing internet connectivity.

### 4.2 Virtual Machine Network

The Windows Server virtual machine is configured to use VMware bridged networking.

This allows the virtual machine to communicate with devices on the physical network, subject to network and firewall configuration.

### 4.3 Connectivity Verification

The following connectivity tests have been performed:

| Test | Result |
|---|---|
| Primary computer to router | Successful |
| Secondary computer to router | Successful |
| Primary computer to secondary computer | Successful |
| Server VM to router | Successful |
| Server VM to physical computers | Successful |

## 5. Planned Lab Architecture

The following systems will be developed throughout the project:

| System | Planned purpose |
|---|---|
| DC01 | Windows Server domain controller |
| CLIENT01 | Windows 11 employee workstation |
| CLIENT02 | Additional Windows client |
| Linux Server | Linux administration and service management |

**Proposed Active Directory domain:** `ad.buttytechnologies.test`

## 6. Infrastructure Safety and Security

The following precautions will be followed:

- Do not publish passwords, private keys, tokens, or credentials.
- Do not expose sensitive configuration information in public documentation.
- Do not run a second DHCP server on the household network.
- Create VMware snapshots before major configuration changes.
- Use fictional company accounts and employee information.
- Verify firewall and network settings before enabling services.

## 7. Current Status

The virtualization environment has been installed, the Windows Server virtual machine has been created, and basic network connectivity has been established.

The next stage is to verify the server's network configuration and prepare it for Windows Server infrastructure services.

---

**Project:** IT Systems Administration Home Lab  
**Simulated Organization:** Butty Technologies
