---
title: "Building a Scalable Home Lab: A Strategic Migration"
date: 2025-08-04
categories: [Virtualization, Networking, Home-Lab, Design]
tags: [introduction, planning, migration, enterprise, hardware, proxmox]
---

## Series Introduction

This comprehensive guide series documents my strategic approach to migrating and modernizing my home lab infrastructure from resource-constrained hardware to a robust, enterprise-aligned environment. The project demonstrates practical application of system administration principles, infrastructure planning, and technical documentation skills essential for IT support and system administration roles.

## The Current Environment & Its Limitations

My existing home lab centers around a Windows Server 2022 Active Directory Domain Controller running as a VM on an older Proxmox host with limited resources. While f  unctional for basic AD operations, this setup presents several critical bottlenecks:

### Performance Constraints:
- Insufficient RAM allocation limiting concurrent VM operations
- Storage I/O bottlenecks affecting domain controller response times
- CPU limitations preventing deployment of additional enterprise services

### Scalability Issues:
- Unable to host multiple VMs simultaneously without performance degradation
- Limited capacity for implementing hybrid cloud scenarios (Azure AD Connect)
- Insufficient resources for network security appliances (pfSense)

### Enterprise Simulation Limitations:
- **Single-host dependency** creates availability risks that don't reflect enterprise redundancy practices
- **Limited service diversity** prevents realistic simulation of multi-tier application environments
- **Insufficient client management capacity** restricts testing of Group Policy, WSUS, and help desk scenarios
- **Network segmentation constraints** limit implementation of proper DMZ, guest network, and security zone configurations
- **Performance bottlenecks** prevent realistic simulation of concurrent user loads and service interactions

The current environment, while functional for basic Active Directory operations, cannot adequately simulate the complex, multi-service enterprise environments that IT support professionals encounter daily. This migration addresses these limitations by establishing a foundation capable of supporting comprehensive business technology scenarios.

> This migration project addresses real-world scenarios IT professionals encounter when scaling infrastructure to meet growing business demands while maintaining service availability. The enhanced capacity enables realistic simulation of small business environments with proper network segmentation, security controls, and service redundancy.
{: .prompt-info }

## The Strategic Solution: Dell Precision 7710 Migration

| Component   | Current Setup                | Upgraded Setup          | Improvement                       |
| ----------- | ---------------------------- | ----------------------- | --------------------------------- |
| Storage     | 240GB SSD                    | 1TB NVMe SSD            | **4x capacity + 3-5x faster I/O** |
| VM Capacity | Single VM at 49% utilization | Multiple concurrent VMs | **5+ VMs simultaneously**         |

### Current Resource Allocation vs. Available Capacity

**Current Host Limitations:**
- **VM Allocation:** 2 vCPUs, 3.78GB RAM (49% of total memory)
- **Remaining Headroom:** 3.92GB RAM (~51% available)
- **Practical Constraint:** Insufficient resources for additional enterprise VMs

**Dell Precision 7710 Projected Capacity:**
- **Same VM Resource Usage:** <15% of total capacity
- **Available for Expansion:** ~27GB RAM, 6 additional vCPUs
- **Storage Allocation:** 1TB NVMe provides ample space for multiple VMs, snapshots, and backup retention
- **Concurrent VM Support:**
  - pfSense VM (1 vCPU, 2GB RAM)
  - Azure AD Connect VM (2 vCPUs, 4GB RAM)
  - Additional lab environments Hardware Specifications Comparison

| Component | Current Proxmox Host | Dell Precision 7710 | Improvement Factor |
|-----------|---------------------|---------------------|-------------------|
| **CPU** | Intel i3-2350M @ 2.30GHz | Intel i7-6920HQ @ 2.60GHz | 4 cores vs 2 cores |
| **Architecture** | 2 cores, 4 threads | 4 cores, 8 threads | **2x cores, 2x threads** |
| **CPU Generation** | 2nd Gen (Sandy Bridge, 2011) | 6th Gen (Skylake, 2015) | **4 generations newer** |
| **Total RAM** | 7.7GB DDR3 | 32GB DDR4 | **4.2x memory capacity** |
| **Memory Type** | DDR3-1333 | DDR4-2133/2400 | **~80% faster memory** |
| **Storage**        | 240GB SSD                    | 1TB NVMe SSD (upgraded)      | **4x storage capacity**  |
### Performance Impact Analysis

| Metric | Current State | Expected Improvement |
|--------|---------------|---------------------|
| **VM Boot Time** | Baseline measurement needed | Faster with NVMe storage |
| **Domain Controller Response** | 4.18% CPU utilization | More responsive with better CPU |
| **Concurrent Operations** | Single VM constraint | Multiple VMs without performance degradation |
| **Storage I/O** | 240GB SSD SATA limitations | 1TB NVMe performance gains |
| **Memory Pressure** | 41.94% utilization in constrained environment | Ample memory for caching and performance |

*Note: Baseline performance metrics from current environment documented in [Guide 1, Figure 8](/posts/ad-migration-bkup). Post-migration performance comparison will be documented in Guide 4 after the Proxmox installation with actual measured improvements.*

### Strategic Advantages:
- **Performance:** Significant CPU and memory improvements enabling concurrent VM operations
- **Storage Expansion:** 4x storage capacity increase (256GB to 1TB NVMe) enabling multiple VM storage without space constraints
- **Scalability:** Sufficient resources for hybrid Azure AD implementations and multiple service VMs
- **Industry Alignment:** Hardware specifications matching entry-level server environments
- **Future-Proofing:** Expansion capabilities for additional learning scenarios

## Enhanced Lab Objectives

This migration enables alignment with enterprise IT best practices and advanced learning opportunities:

### Immediate Goals
1. **Zero-Downtime Migration:** Seamlessly migrate existing AD DC with minimal service interruption
2. **Performance Optimization:** Achieve measurable improvements in domain services response times
3. **Infrastructure Modernization:** Implement proper VLAN segmentation reflecting enterprise standards

### Advanced Capabilities
1. **Hybrid Cloud Integration:** Deploy Azure AD Connect for hybrid identity management
2. **Network Security:** Implement pfSense for enterprise-grade routing, firewalling, and VPN services
3. **Client Management:** Establish dedicated environment for testing Group Policy and system management scenarios

## Network Infrastructure Evolution

The migration includes strategic network redesign to align with enterprise best practices:

### Current VLAN-to-Port Mapping:
- VLAN 11 (Operations): Port 2
- VLAN 20 (Accounting): Ports 3-4
- VLAN 30 (Tech Support): Port 5 ← *AD DC currently here*
- VLAN 40 (Marketing): Port 8
- VLAN 50 (Management): Ports 6-7
- VLAN 60 (Server Room): Port 10


![Current Network Topology](/assets/img/posts/dell-vm-upgrade-intro/dell-vm-upgrade-intro-img1.jpg)
*Figure 1: Current Network Topology - Existing infrastructure utilizing Cudy GS1016E managed switch with departmental VLAN segmentation. The Windows Server 2022 AD DC currently resides on VLAN 30 (Tech Support) creating service location misalignment with enterprise best practices.*

### Planned VLAN Role Transitions:
- **VLAN 30:** Tech Support → Server VLAN/Server Closet
- **VLAN 60:** Server Room → Tech Support (192.168.60.0/28)

![Planned Network Topology After Upgrade](/assets/img/posts/dell-vm-upgrade-intro/dell-vm-upgrade-intro-img2.jpg)
*Figure 2: Planned Network Topology - Post-migration architecture showing VLAN 30 transition to dedicated Server VLAN role hosting Dell Precision 7710 with multiple VMs, and VLAN 60 repurposed for Tech Support functions with proper network segmentation.*

> **Warning Box:** VLAN restructuring will be performed after AD migration stabilization to minimize risk of service disruption during critical infrastructure changes.
{: .prompt-warning }

## Series Roadmap & Learning Outcomes

This project is documented through interconnected guides, each focusing on specific technical competencies:

### Phase 1: Foundation & Migration

**Guide 2: Installing Proxmox VE and Initial Network Configuration on Dell Precision 7710**
- Document bare-metal hypervisor installation and storage configuration
- Configure initial network setup for Proxmox management and pfSense preparation
- Set up NVMe SSD for optimized VM storage performance
- Address pre-migration dcdiag issues:
  - **SystemLog failure** on homelab-ad domain controller
  - **DNS resolution timeouts** for ldap._tcp.dc._msdcs.homelab.lan
  - **Dynamic DNS registration failures** for homelab.lan A records
  - **NtpClient warnings** requiring external time source configuration for PDC Emulator

### Phase 2: Network Infrastructure

**Guide 3: Deploying and Configuring pfSense as Primary Firewall and Router**
- Create pfSense VM with dual-NIC configuration (integrated RJ45 and USB-to-Ethernet)
- Configure WAN/LAN interfaces, DHCP services, and DNS forwarding
- Implement firewall rules and network segmentation for lab environment security

### Phase 3: Service Migration & Integration

**Guide 4: Migrating and Integrating Active Directory Domain Controller**
- Restore AD DC VM from backup with hardware-optimized resource allocation
- Configure network integration with pfSense-controlled segments
- Perform comprehensive AD DS functionality verification
- Document performance improvements with before/after metrics comparison

### Phase 4: Advanced Services

**Guide 5: Implementing Hybrid Identity with Azure AD Connect**
- Deploy dedicated Windows Server 2022 VM for hybrid identity services
- Configure Azure AD Connect for on-premises to cloud identity synchronization
- Test hybrid authentication and directory synchronization functionality

## Technical Skills Demonstrated

This series showcases practical application of skills directly relevant to IT support and system administration roles:

### Infrastructure Management
- **Virtualization:** Proxmox hypervisor deployment, VM migration, and resource optimization
- **Active Directory:** Domain controller management, health monitoring, and performance tuning
- **Network Administration:** VLAN configuration, routing protocols, and security implementation

### Enterprise Technologies
- **Cloud Integration:** Hybrid Azure AD setup and directory synchronization
- **Security Implementation:** pfSense firewall deployment and network segmentation
- **Performance Analysis:** Resource utilization monitoring and capacity planning

### Professional Practices
- **Documentation:** Technical writing and process documentation for enterprise environments
- **Risk Management:** Backup strategies, rollback planning, and change management procedures
- **Project Management:** Systematic approach to infrastructure upgrades with minimal downtime

## Project Success Metrics

Each guide includes measurable outcomes demonstrating technical proficiency:

- **Performance Improvements:** Quantified response time enhancements post-migration
- **Service Availability:** Documentation of uptime maintenance during migration process
- **Security Posture:** Network segmentation effectiveness and access control implementation
- **Scalability Achievement:** Successful deployment of multiple concurrent enterprise services

> This systematic approach to infrastructure projects demonstrates the methodical planning and execution skills valued in professional IT environments, directly applicable to help desk escalation, system administration, and network support scenarios.
{: .prompt-tip }

## Conclusion

This home lab migration project represents more than hardware upgrades—it demonstrates strategic thinking, technical execution, and documentation skills essential for modern IT professionals. Each guide in this series provides practical insights applicable to real-world infrastructure projects while showcasing hands-on experience with enterprise technologies.

The migration from resource-constrained hardware to a robust, scalable environment enables advanced learning opportunities and aligns my home lab with industry best practices. This foundation supports continued professional development and demonstrates readiness for IT support and system administration responsibilities.

---

*Next in Series: Guide 2 - "Installing Proxmox VE and Initial Network Configuration on Dell Precision 7710"*
