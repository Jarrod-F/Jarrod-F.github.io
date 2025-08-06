---
title: "Series Introduction – Building a Scalable Home Lab: A Strategic Migration"
date: 2025-08-04
categories: [Virtualization, Networking, Home-Lab, Design]
tags: [introduction, planning, migration, enterprise, hardware, proxmox]
---

## Series Introduction

This comprehensive guide series documents the strategic approach to migrating and modernizing my home lab infrastructure from resource-constrained hardware to a robust, enterprise-aligned environment. The project demonstrates practical application of system administration principles, infrastructure planning, and technical documentation skills essential for IT support and system administration roles.

---

## The Current Environment & Its Limitations

The existing home lab centers around a Windows Server 2022 Active Directory Domain Controller running as a VM on an older Proxmox host with limited resources. While functional for basic AD operations, this setup presents several critical bottlenecks that prevent realistic enterprise simulation.

**Key Limitations:**
* **Performance Constraints:** Insufficient RAM, storage I/O bottlenecks, and CPU limitations prevent the deployment of additional enterprise services and affect domain controller response times.
* **Scalability Issues:** The system is unable to host multiple VMs simultaneously without significant performance degradation, limiting capacity for hybrid cloud scenarios or network security appliances like pfSense.
* **Enterprise Simulation Gaps:** A single-host dependency creates availability risks, and limited service diversity prevents the realistic simulation of multi-tier application environments, concurrent user loads, or proper network segmentation (e.g., DMZ, guest networks).

> **The Core Problem**
>
> The current environment, while functional for basic Active Directory operations, cannot adequately simulate the complex, multi-service enterprise environments that IT support professionals encounter daily. This migration addresses these limitations by establishing a foundation capable of supporting comprehensive business technology scenarios.
{: .prompt-info}

---

## The Strategic Solution: Dell Precision 7710 Migration

The core of the solution is a hardware migration to a significantly more powerful Dell Precision 7710 workstation.

### Hardware Specifications Comparison

| Component          | Current Proxmox Host         | Dell Precision 7710       | Improvement Factor    |
| :----------------- | :--------------------------- | :------------------------ | :-------------------- |
| **CPU**            | Intel i3-2350M @ 2.30GHz     | Intel i7-6920HQ @ 2.60GHz | 4 cores vs 2 cores    |
| **Architecture**   | 2 cores, 4 threads           | 4 cores, 8 threads        | 2x cores, 2x threads  |
| **CPU Generation** | 2nd Gen (Sandy Bridge, 2011) | 6th Gen (Skylake, 2015)   | 4 generations newer   |
| **Total RAM**      | 7.7GB DDR3                   | 32GB DDR4                 | 4.2x memory capacity  |
| **Memory Type**    | DDR3-1333                    | DDR4-2133/2400            | ~80% faster memory    |
| **Storage**        | Traditional HDD/SSD          | NVMe SSD (planned)        | 3-5x faster I/O       |
| **VM Capacity**    | Single VM at 49% util.       | Multiple concurrent VMs   | 5+ VMs simultaneously |

### Performance Impact Analysis

| Metric                    | Current State                          | Expected Improvement                         |
| :------------------------ | :------------------------------------- | :------------------------------------------- |
| **VM Boot Time**          | Baseline measurement needed            | Faster with NVMe storage                     |
| **DC Response**           | 4.18% CPU utilization                  | More responsive with better CPU              |
| **Concurrent Operations** | Single VM constraint                   | Multiple VMs without performance degradation |
| **Storage I/O**           | HDD/SATA limitations                   | NVMe performance gains                       |
| **Memory Pressure**       | 41.94% utilization in constrained env. | Ample memory for caching and performance     |

> **Note:** Baseline performance metrics from the current environment were documented in [Guide 1: Backing Up an Active Directory VM for Migration](/posts/ad-migration-bkup). Post-migration performance comparison will be documented in guide 4 Migrating and Integrating Active Directory, with actual measured improvements.
{: .prompt-info}

---

## Enhanced Lab Objectives

This migration enables alignment with enterprise IT best practices and advanced learning opportunities.

### Immediate Goals
* **Zero-Downtime Migration:** Seamlessly migrate the existing AD DC with minimal service interruption.
* **Performance Optimization:** Achieve measurable improvements in domain services response times.
* **Infrastructure Modernization:** Implement proper VLAN segmentation reflecting enterprise standards.

### Advanced Capabilities
* **Hybrid Cloud Integration:** Deploy Azure AD Connect for hybrid identity management.
* **Network Security:** Implement pfSense for enterprise-grade routing, firewalling, and VPN services.
* **Client Management:** Establish a dedicated environment for testing Group Policy and system management scenarios.

---

## Network Infrastructure Evolution

The migration includes a strategic network redesign to align with enterprise best practices by transitioning VLAN 30 to a dedicated "Server VLAN" role.

> **Warning: Phased Implementation**
>
> This VLAN restructuring will be performed *after* the Active Directory migration is complete and stable to minimize the risk of service disruption during critical infrastructure changes.
{: .prompt-warning}

**Current VLAN-to-Port Mapping:**
* **VLAN 11 (Operations):** Port 2
* **VLAN 20 (Accounting):** Ports 3-4
* **VLAN 30 (Tech Support):** Port 5 ← *AD DC currently here*
* **VLAN 40 (Marketing):** Port 8
* **VLAN 50 (Management):** Ports 6-7
* **VLAN 60 (Server Room):** Port 10

![Current Network Topology Diagram](/assets/img/posts/dell-vm-upgrade-intro/dell-vm-upgrade-intro-img1.jpg)
*Figure 2: Post-migration architecture showing VLAN 30 transitioned to a dedicated Server VLAN role, hosting the Dell Precision 7710 with multiple VMs.*

**Planned VLAN Role Transitions:**
* **VLAN 30:** Tech Support → **Server VLAN / Server Closet**
* **VLAN 60:** Server Room → **Tech Support** (`192.168.60.0/28`)

![Planned Network Topology Diagram](/assets/img/posts/dell-vm-upgrade-intro/dell-vm-upgrade-intro-img2.jpg)
*Figure 2: Post-migration architecture showing VLAN 30 transitioned to a dedicated Server VLAN role, hosting the Dell Precision 7710 with multiple VMs.*

---

## Series Roadmap & Learning Outcomes

This project is documented through interconnected guides, each focusing on specific technical competencies.

### Phase 1: Foundation & Migration
* **Guide 1: Backing Up My On-Prem Active Directory Domain Controller VM for Migration**
    * Pre-migration documentation of existing AD DC environment (IP configuration, hostname, domain health)
    * Critical backup procedures using Proxmox vzdump for enterprise-grade data protection
    * Comprehensive* AD health assessment using dcdiag testing suite
    * Backup integrity verification and recovery planning methodology
    * Risk assessment documentation including downtime planning and rollback strategies
* **Guide 2: Installing Proxmox VE on Dell Precision 7710**
    * Documents bare-metal hypervisor installation, storage configuration, and initial network setup.
* **Addressing Pre-migration Issues:**
    * A plan to resolve the `dcdiag` warnings (SystemLog, DNS timeouts, NTP) noted in Guide 1.

### Phase 2: Network Infrastructure
* **Guide 3: Deploying and Configuring pfSense**
    * Covers creating the pfSense VM, configuring WAN/LAN interfaces, and implementing firewall rules.

### Phase 3: Service Migration & Integration
* **Guide 4: Migrating and Integrating Active Directory**
    * Details restoring the AD DC VM, network integration, and a full performance comparison.

### Phase 4: Advanced Services
* **Guide 5: Implementing Hybrid Identity with Azure AD Connect**
    * Focuses on deploying a dedicated server for hybrid identity and testing synchronization.

> **Tip: Demonstrating Professional Skills**
>
> This systematic approach to infrastructure projects demonstrates the methodical planning, execution, and documentation skills valued in professional IT environments, directly applicable to help desk escalation, system administration, and network support scenarios.
{: .prompt-tip}

## Conclusion

This home lab migration project represents more than a hardware upgrade—it demonstrates strategic thinking, technical execution, and documentation skills essential for modern IT professionals. Each guide in this series provides practical insights applicable to real-world infrastructure projects while showcasing hands-on experience with enterprise technologies.

**Next in Series:** [Guide 1: Backing Up an Active Directory VM for Migration](/posts/ad-migration-bkup)
