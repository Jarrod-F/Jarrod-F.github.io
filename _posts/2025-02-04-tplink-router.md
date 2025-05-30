---

title: Configuring a TP-Link Router for a Home Lab Network
date: 2025-02-05 
categories: [Networking, Home Lab, Academic-Project]
tags: [tp-link, router, vlan, omada, managed switch, configuration, tutorial]

---

> *Project Origin: This project was undertaken as part of the "Special LAN Projects" elective during my academic program. It was a self-directed endeavor involving the complete lifecycle from network design and hardware selection, component upgrades, configuration, troubleshooting, and comprehensive documentation.*
{: .prompt-info }

## Introduction

This post details the steps I took to configure and set up a TP-Link router for my home lab network. It covers the initial setup, VLAN configuration, and integration with both a Xnyahitog XY-M2F8P managed switch and the primary ISP (XB7-t) router, **sharing insights from the challenges faced and solutions implemented.**

An additional TP-Link router was essential for achieving the home lab objectives because the ISP-provided router has restricted configuration access. The ISP router acts as the primary internet gateway for both the regular home network and the lab network, connecting to the TP-Link router's **WAN port**. The TP-Link router then serves as the central gateway for the internal lab network (LAN), while the Xnyahitog managed switch handles traffic distribution to various lab devices using VLANs.

This segmented setup provides the added benefit of isolating the home lab network, protecting the main home network from potential connectivity issues arising from experiments and changes within the lab environment.

Before starting the router configuration, I installed the **Omada Controller Software Interface (OC)**. This provides more granular control over the router setup and enables cloud access for managing the network remotely. The Xnyahitog XY-M2F8P managed switch was connected to the network initially without any specific VLAN configurations applied until later in the process (see Configuration Step #5).

---

## Initial Setup

1.  **Planning:**
    * Reviewed the manufacturer's manual and support videos to formulate an overall configuration plan before starting.
2.  **Cabling:**
    * Measured and cut required lengths of network cabling.
    * Attached RJ45 connectors to the cables.
    * Verified each cable connection using a cable tester before proceeding.
3.  **Connecting to ISP Router:**
    * Connected an RJ45 cable from an available LAN port on the ISP router to the primary **WAN port** on the TP-Link router.
    * Confirmed the connection status appeared as "Connected" within the Omada Controller (OC) interface.
4.  **Labeling the WAN Port:**
    * In the OC, navigated to **Wired Networks > Internet**.
    * Labeled the description tag for the newly connected WAN port as "ISP (XFINITY) XB7-t". This adheres to networking documentation best practices and helps ensure clarity for anyone else who might need to troubleshoot or modify router settings later. See Figure 1 for an example.

    ![OC WAN Port Labeling Example](/assets/img/posts/tplink-router/tplink-router-1.png)
    *Figure 1: Example of labeling the WAN port in Omada Controller.*

---

## Configuration Steps

1.  **Adding VLANs:**
    * Began adding VLANs according to the network subnet plan. This involved defining the corresponding network address, gateway, subnet mask, and DHCP range for each VLAN within the router configuration.
    * Initially created **VLAN 10** using the LAN interface `WAN/LAN2`, as shown conceptually in Figure 2, and saved the configuration.

      - **Note:** After completing steps 1-6 Note: After initial configuration phases and encountering testing issues (detailed in Testing Techniques > Troubleshooting > #1), VLAN 10 was ultimately deleted and replaced with **VLAN 11** due to connectivity issues encountered during testing.

    * Inside the OC, configured the **"Interfaces"** option rather than "VLAN" as the "Purpose" setting, following guidance from manufacturer support resources. This is necessary to assign the subnet, DHCP range, and other required network parameters.
    * Other optional features were left unconfigured at this stage to first establish and verify basic functionality.

    ![Example VLAN Configuration Interface](/assets/img/posts/tplink-router/tplink-router-2.png)
    *Figure 2: Conceptual view of the VLAN configuration interface.*

2.  **Changing LAN Port Assignment:**
    * After initially assigning port `WAN/LAN2` as the LAN interface for VLAN 10, I decided to use `LAN1` Port instead. The physical location of `LAN1` on the router better suited the cabling layout for connecting devices.
    * Edited the **VLAN 10** configuration, changed the interface setting to `LAN1`, saved the changes, and proceeded to configure the remaining VLANs. Figure 3 shows an overview page reflecting the completed VLAN setup.

    ![Overview of Configured VLANs](/assets/img/posts/tplink-router/tplink-router-3.png)
    *Figure 3: Overview page showing all configured VLANs.*

3.  **Addressing Port Connectivity Issues:**
    * This change revealed connectivity issues later When switchingthe trunk cable from the router's `WAN/LAN2` port to the `LAN1` port, all VLANs had connectivity *except* VLAN 10, despite the interface setting having been updated in the router (see Testing Techniques > Troubleshooting #1).

4.  **Reviewing Switch Profile Settings:**
    * Next, reviewed the settings under **Wired Networks > LAN > Switch Profile**. This page contains the VLAN Tag settings for switch ports managed by the Omada Controller.
    * When the VLANs were created (in Step #1), corresponding "Switch Profiles" were automatically generated by the router.
    * *Problem Identified:* The auto-generated profiles for the VLANs had their **Native Network** set to **Default(1)** (indicating the trunk port setting) and the **Untagged Networks** included *both* the corresponding VLAN ID (e.g., 10, 20, 30) *and* **Default(1)**. This default configuration (shown conceptually in Figure 4) is incompatible with the intended two-router and switch setup for correct traffic routing.

    ![Auto-Configured Incorrect Switch Profile Settings](/assets/img/posts/tplink-router/tplink-router-4.png)
    *Figure 4: Example of incompatible auto-configured switch profile settings.*

    * *Explanation:* The **Native Network** determines the Port VLAN Identifier (PVID) for a switch port. When a port receives an untagged frame, the switch adds a VLAN tag based on the PVID and forwards the frame within that native network. Therefore, for access ports, the Native Network should be set to the specific VLAN's PVID, not Default(1). Similarly, the **Untagged Networks** setting for an access port should typically only include the specific VLAN ID it belongs to, not Default(1).
    * *Solution:* It was not possible to save the necessary changes by directly editing the auto-created switch profiles. Therefore, I had to **delete all the auto-generated profiles** and manually create a **new profile for each VLAN** with the correct settings (Native Network = VLAN ID, Tagged Networks = None, Untagged Networks = VLAN ID). See Figure 5 for an example of correctly configured settings for a VLAN profile.

    ![Correctly Configured Switch Profile Settings Example](/assets/img/posts/tplink-router/tplink-router-5.png)
    *Figure 5: Example of correctly configured switch profile settings.*

5.  **Integration with Managed Switch:**
    * After completing the router's VLAN and switch profile configurations, the Xnyahitog XY-M2F8P managed switch was configured with the corresponding VLANs. This aligned the switch with the network plan designed to isolate different "departments" (represented by VLANs) for security while still allowing necessary inter-VLAN communication. (Refer to the separate Xnyahitog XY-M2F8P managed switch Configuration and Testing Guide for details on the switch setup).

---

## Testing Techniques

### General Testing Guidelines

1.  **Router Configuration:** Access the router's web interface (via Omada Controller in this case) from a computer directly connected to a LAN port (initially, the default LAN might be easiest before VLANs are fully active). Complete the necessary settings: VLANs, DHCP scopes, Inter-VLAN Routing rules, etc.
3.  **Connecting Computers After Configuration:** *Only after* completing the configuration on both the router and the switch should you connect end devices (computers, servers, etc.) to their designated VLAN ports on the switch. This methodical approach helps isolate potential issues to configuration steps rather than complicating troubleshooting with potential cabling or hardware faults.
2.  **Switch Configuration:** Access the managed switch’s web interface from a computer directly connected to one of its ports. Configure the corresponding VLANs and assign these VLANs to specific switch ports (access ports). Set up the trunk port (connecting to the router) to carry tagged traffic for all VLANs, as outlined in the [Xnyahitog XY-M2F8P switch tutorial](/posts/industrial-switch).
4.  **Perform Connectivity Tests:**
    * **Ping Tests:** Test connectivity between computers on the *same* VLAN and then between computers on *different* VLANs (to verify inter-VLAN routing).
    * **Network Services:** Verify that DHCP is correctly assigning IP addresses, subnet masks, and gateway information appropriate for each computer's assigned VLAN.
    * **Internet & Inter-VLAN Access:** Ensure devices on each VLAN can access the internet (via the router) and communicate with devices on other VLANs as permitted by the configuration.
    * **Useful PowerShell and Command Prompt Commands Used:**
        * `ipconfig /all`: Verify IP address, subnet mask, gateway, DNS, and MAC address on the test PC.
        * `ping <IP_address_or_hostname>`: Test basic layer 3 connectivity to other devices, the gateway, or internet addresses.
        * `tracert <IP_address_or_hostname>`: Trace the network path packets take to a destination, helpful for diagnosing routing loops or failures.
        * `netsh`: A powerful command-line utility for viewing and configuring various network settings.
            * `netsh winsock reset`: Resets the Winsock Catalog to a clean state (often requires reboot).
            * `netsh int ip reset`: Resets the TCP/IP stack (often requires reboot).
            * `netsh advfirewall reset`: Resets the Windows Firewall configuration to default.
        * `Restart-NetAdapter -Name "Ethernet 2"` (PowerShell): Restarts a specific network adapter. Replace `"Ethernet 2"` with the actual name.
        * `Restart-NetAdapter -Name "E*2"` (PowerShell): Restarts network adapters matching a wildcard pattern.
        * `ipconfig /flushdns`: Clears the local DNS resolver cache.
        * `ipconfig /release`: Releases the current DHCP-assigned IP address lease.
        * `ipconfig /renew`: Requests a new IP address lease from the DHCP server.

### Troubleshooting

1.  **VLAN 10 Connectivity Issues:**
    * *Symptoms:* Devices connected to switch ports assigned to VLAN 10 could not obtain an IP address via DHCP or communicate on the network after the `LAN1` port change described in Configuration Step #2 & #3.
    * *Steps Taken:*
        * Double-checked VLAN settings (ID, subnet, gateway, DHCP range), interface assignments, and switch profile configurations on the router (Omada Controller).
        * Verified corresponding VLAN settings and port assignments (access port PVID, trunk port tagging) on the managed switch.
        * Rebooted both the switch and the router multiple times.
        * Deleted and recreated the VLAN 10 settings and its associated switch profile on the router.
        * Conducted extensive online research for similar issues with TP-Link Omada setups.
        * Contacted TP-Link support, who ultimately suggested a factory reset of the router.
  
    * *Solution:* Before resorting to a factory reset, I attempted a last troubleshooting step:
        *  Deleted **VLAN 10** configuration entirely from *both* the router and the switch.
        * Recreated the VLAN using a new ID (VLAN 11) and reassigned interface roles.
        * Configured the corresponding switch profile for VLAN 11 on the router and the necessary VLAN settings (port assignments, tagging) on the switch.
        * Rebooted both devices.
         

This resolved the connectivity issue, suggesting a persistent configuration glitch related specifically to the VLAN ID 10 after the initial port assignment change. **This experience underscored a valuable lesson:** before resorting to a full device factory reset and losing all configurations, when standard troubleshooting for a specific element (like recreating VLAN 10 with the same ID) repeatedly fails, a more targeted yet fundamental change—like altering the problematic ID itself (e.g., to VLAN 11)—can be a highly effective intermediate step.

It highlighted that network device configurations can occasionally retain 'ghost' settings or states that aren't cleared by simple reconfigurations of the same identifier, and that forcing the use of a new ID can circumvent this without requiring a complete system wipe, thereby saving significant reconfiguration time.

---

## Next Steps and Future Implementations

The TP-Link router offers numerous features to enhance network security and performance. Based on the initial setup, my planned next steps involve implementing and testing the following:

* **Access Control List (ACL):** Configure ACLs for more granular control over which devices or subnets can access specific network resources, adding a layer of security beyond basic VLAN segregation.
* **URL Filtering:** Implement URL filtering to block access to specific websites, potentially improving security and productivity within the lab environment.
* **MAC Filtering:** Configure MAC filtering to allow only devices with explicitly permitted MAC addresses to connect to the network (or specific VLANs), adding another barrier against unauthorized access.

![Security Features Menu Example](/assets/img/posts/tplink-router/tplink-router-6.png)
*Figure 6:Tp-Link router some of the advanced security features.*

Additionally, after strengthening the security posture, I plan to explore and test specific features within the VLAN settings further:

* **IGMP Snooping:** Enable and configure IGMP Snooping to optimize multicast traffic management. This can improve network efficiency, especially if using applications that rely on multicast, by preventing multicast data from flooding ports unnecessarily.

Finally, I will continue exploring other advanced features related to network security, transmission optimization, and potentially VPN configurations to further refine the home lab network's performance, security, and capabilities.

I hope you found this useful, if you have any questions, feel free to reach out to me on LinkedIn.
