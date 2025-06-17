---
title: Active Directory Configuration on Windows Server 2022
date: 2025-05-26 
categories: [Active Directory, Windows Server, Home Lab, Academic-Project]
tags: [activedirectory, ad-ds, dns, gpo, ou, domain-controller, server-2022, configuration]
---

> **Context:** This guide documents my end-to-end process for setting up Active Directory Domain Services (AD DS), DNS, Organizational Units, Group Policies, and file permissions in a home lab—complete with challenges faced and solutions implemented.
{: .prompt-info}

## Introduction

After installing Microsoft Server 2022 and configuring its network settings, I proceeded to set up Active Directory (AD) to manage user accounts, groups, and resources. This involved:

- Installing and configuring AD DS and DNS.
- Creating Organizational Units (OUs) and user accounts.
- Designing and applying Group Policies (GPOs).
- Securing file and folder permissions.

Throughout, I documented troubleshooting steps to showcase my problem-solving approach.

---

## Initial Steps & Planning

Before installing Active Directory Domain Services (AD DS), it’s essential to ensure that your DNS configuration is correctly set up. Computers joining the domain must be able to locate your Windows Server acting as the domain controller. This can be achieved by setting your server as the primary DNS for your network or by configuring DNS forwarding on your primary DNS server specifically for the Active Directory domain.

I also considered the impact of these changes on my devices and overall network when my server trial period ends. I researched and created the plan outlined below before beginning.

### Ensuring Smooth Transition After Server Trial Ends

> **Tip:** If using the server trial version for learning purposes only, plan ahead for role reversal when the trial expires.
{: .prompt-tip}

1.  **Plan for Role Reversal:**
    * Identify server roles (DNS, DHCP, AD DS), document current settings, and plan to revert these functions to their original configuration (e.g., on the router) once the trial ends.
    * **DNS and DHCP:** Ensure the router or another device can take over these roles. Document the current DNS and DHCP settings on the TP-Link ER605 router.
    * **File Shares:** Plan to transfer any critical data off the server before it expires.
2.  **Deactivate Server Roles Gradually:**
    * Reduce dependency on the server before the trial period concludes.
    * **Backup AD DS Data:** Back up important data. Properly demote the server using Server Manager or PowerShell cmdlets before shutting it down. See [Microsoft Windows Server Documentation](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/demoting-domain-controllers-and-domains--level-200-) for detailed instructions.
    * **DNS Records:** Update or remove DNS records on the server as needed.
3.  **Switch Back to Router DHCP/DNS:**
    * Ensure the TP-Link ER605 router is configured to resume DHCP and DNS roles for the network.
4.  **Client Reconfiguration:**
    * Change network adapter settings on client devices to obtain DNS settings automatically (from the router).
5.  **Router Configuration Adjustment:**
    * Change VLAN and DNS server settings on the router from manual (pointing to the Windows Server) back to automatic or its previous state.

---

## Configuration Steps

### 1. Assign the Server a Name

After logging into the Server VM, the **Server Manager** window will display by default. The first step is to assign a name to the server, as changing it later after AD DS installation can be complex.

* In **Server Manager**, click <kbd>Local Server</kbd> in the left pane.
* In the **Properties** section, click the existing computer name link.
* In the **System Properties** pop-up, click <kbd>Change...</kbd>.
* Enter a descriptive server name (e.g., `DC01`).

![Changing Server Name - System Properties](/assets/img/posts/ms-ad-config/ms-ad-config-img1.png)
*Figure 1: System Properties dialog for changing the computer name.*

* Click <kbd>OK</kbd> to apply the changes, then restart the server to update the settings.

![Restart prompt after changing server name](/assets/img/posts/ms-ad-config/ms-ad-config-img2.png)
*Figure 2: Prompt to restart the server.*

### 2. Installing AD DS and DNS Roles

Once the server has restarted and you are logged in again:

* In **Server Manager**, click <kbd>Manage</kbd> → <kbd>Add Roles and Features</kbd>. This will start the **Add Roles and Features Wizard**.
  * Click <kbd>Next</kbd> to begin.

![Add Roles and Features Wizard - Before You Begin](/assets/img/posts/ms-ad-config/ms-ad-config-img3.png)
*Figure 3: The "Before You Begin" screen of the Add Roles and Features Wizard.*


* **Installation Type:** Keep the default "<kbd>Role-based or feature-based installation</kbd>" radio button checked → <kbd>Next</kbd>.
* **Server Selection:** Keep the default "<kbd>Select a server from the server pool</kbd>" checked
  * ensure your server is highlighted, → <kbd>Next</kbd>.
* **Server Roles:** Select the following roles:
    * **Active Directory Domain Services** → click **Add Features**.
    * **DNS Server** → click **Add Features**.
* **Features:** Leave the default features selected → <kbd>Next</kbd>.
* **AD DS / DNS Server:** Click through the informational pages for AD DS and DNS Server.
* **Confirmation:** Review your selections. If changes are needed, use the <kbd>Previous</kbd> button.
  *  Check the box "<kbd>Restart the destination server automatically if required</kbd>". 
  *  Click <kbd>Install</kbd> → close the wizard after completion.

### 3. Promote the Server to a Domain Controller

After installing the roles → promote the server to a domain controller.

* In **Server Manager**, click the flag icon in the top right
  * click the blue link: "<kbd>Promote this server to a domain controller</kbd>".

![Promote server to domain controller link in Server Manager](/assets/img/posts/ms-ad-config/ms-ad-config-img4.png)
*Figure 4: Notification to promote the server.*

### 4. Creating a New Forest and Domain

This will launch the **Active Directory Domain Services Configuration Wizard**.

* **Deployment Configuration:**
    * Options include: 
        * ***Add this domain controller to an existing domain:*** Used for adding more DCs to an existing domain for redundancy/load balancing.
        * ***Add a new domain to an existing forest:*** Used for larger organizations with multiple domains in a hierarchy.
        * ***Add a new forest:*** Creates the first domain in a new AD forest.  

    * **Choose** "<kbd>Add a new forest</kbd>"
    * **Root domain name:** Enter a meaningful root domain, I chose `homelab.lan`.
        > For a home lab, using a non-publicly routable top-level domain like `.lan` or `.local` is common. In a production environment, you would typically use a subdomain of your registered public domain name (e.g., `ad.yourcompany.com`).
        {: .prompt-tip}

![AD DS Configuration Wizard - Add a new forest](/assets/img/posts/ms-ad-config/ms-ad-config-img5.png)
*Figure 5: Naming the new forest and root domain.*

* **Domain Controller Options:**
    * Leave the default Forest and Domain functional levels.
    * Ensure **Domain Name System (DNS) server** and **Global Catalog (GC)** are checked.
    * Enter a strong **Directory Services Restore Mode (DSRM)** password → <kbd>Next</kbd>

      > Warning: This password is critical for AD recovery scenarios. Store it in a safe place.
      {: .prompt-warning}  

![AD DS Configuration Wizard - Domain Controller Options](/assets/img/posts/ms-ad-config/ms-ad-config-img6.png)
*Figure 6: Setting Domain Controller options and DSRM password.*

* **DNS Options:**
    * A warning regarding DNS delegation may appear. You can safely ignore this in a home lab environment by clicking <kbd>Next</kbd>.
        > Tip: In production enviornment, a dedicated DNS server and DNS delegation would be used.
        {: .prompt-tip}
* **Additional Options:**
    * Leave the **NetBIOS domain name** as the default suggested by the wizard (e.g., `HOMELAB`) → <kbd>Next</kbd>.
* **Paths:**
    * Leave the default folder paths for the AD DS database, log files, and SYSVOL. → <kbd>Next</kbd>.
* **Prerequisites Check:** The system will perform a prerequisites check. Warnings are normal in a home lab setup. Click <kbd>Install</kbd> to begin the promotion.

![AD DS Configuration Wizard - Prerequisites Check](/assets/img/posts/ms-ad-config/ms-ad-config-img7.png)
*Figure 7: Prerequisites check before installation.*

* The server will restart automatically after the installation and promotion are complete.

### 5. Setting up DNS Forwarding (DNS Proxy)

To allow clients in your domain to resolve both internal AD resources (like `homelab.lan`) and external internet addresses, DNS needs to be configured correctly.

* **Option A: Windows Server as Primary DNS for all clients:** You can configure your DHCP server (likely your router) to assign the Windows Server's IP address as the primary DNS server to all clients. In this case, you'd configure "Forwarders" in the DNS Server role on Windows Server to point to external DNS servers (like your ISP's DNS or public ones like `9.9.9.9`).
    > If the Windows Server (acting as the sole DNS) goes offline or is undergoing updates, clients may lose internet connectivity. This setup is common in businesses for internal resolution but often involves multiple resilient DNS servers.
    {: .prompt-warning}

* **Option B: DNS Forwarding on the Router (My Choice):** To avoid potential internet connectivity issues if my single Windows Server VM was offline, I configured DNS forwarding (sometimes called conditional forwarding or DNS proxy) on my TP-Link router.
    * The settings were found under <kbd>Wired Networks</kbd> → <kbd>LAN</kbd> → (Edit icon for each VLAN).
    * For **DNS Server**, I chose the manual setting:
        * **Primary DNS:** Server IP (e.g., `192.168.30.190` - *Note: Ensure you use the Windows Server's IP, not the Proxmox host's IP.*)
        * **Secondary DNS:** An external DNS server (e.g., Quad9 - `9.9.9.9`) for fallback.
 

![Client nslookup test for domain resolution](/assets/img/posts/ms-ad-config/ms-ad-config-img8.png)
*Figure 8: DNS forward settings in Tp-Link Router.*

  * Next, I verified DNS functionality using `nslookup homelab.lan` on a client device.

  * On client PCs, the network adapter settings were configured to use these DNS servers (or receive them via DHCP if the router was set to distribute them for that VLAN).

![Client Network Adapter DNS Settings](/assets/img/posts/ms-ad-config/ms-ad-config-img9.png)
*Figure 9: Example of client network adapter DNS settings in Windows 11 OS.*

  * On clients, I used `ipconfig /flushdns` and `ipconfig /registerdns` to quickly apply the new settings via the command line.

### 6. Verifying and Configuring DNS Settings on the Server

My initial setup in step 5 did not immediately result in proper DNS resolution.

* **Forward Lookup Zone:** In **DNS Manager** on the server (accessible from Server Manager > Tools), I confirmed that the `homelab.lan` forward lookup zone existed and its properties were correct.
  > Tip: Reverse lookup zone must be configured on server for proper DNS functionality in the home lab configuration.
  {: .prompt-tip}
* **Reverse Lookup Zone:** I noticed there was no entry in **Reverse Lookup Zones**. This zone is crucial for mapping IP addresses back to hostnames, essential for some network services and troubleshooting.
    * To create it:
        1.  Open **DNS Manager**.
        2.  Expand the server node, right-click on <kbd>Reverse Lookup Zones</kbd> → <kbd>New Zone...</kbd>.
        3.  Follow the wizard: select "<kbd>Primary Zone</kbd>", choose the storage option (Active Directory-integrated), select "<kbd>IPv4 Reverse Lookup Zone</kbd>", enter the Network ID part of your server's subnet (e.g., `192.168.30`), and enable secure dynamic updates.

![DNS Manager - Creating New Reverse Lookup Zone (Network ID)](/assets/img/posts/ms-ad-config/ms-ad-config-img10.png)
*Figure 10: Specifying Network ID for Reverse Lookup Zone.*

![DNS Manager - Creating New Reverse Lookup Zone (Zone Name)](/assets/img/posts/ms-ad-config/ms-ad-config-img11.png)
*Figure 11: Auto-generated zone name for Reverse Lookup Zone.*

![DNS Manager - Creating New Reverse Lookup Zone (Dynamic Update)](/assets/img/posts/ms-ad-config/ms-ad-config-img12.png)
*Figure 12: Setting Dynamic Update option.*

![DNS Manager - Completing New Reverse Lookup Zone Wizard](/assets/img/posts/ms-ad-config/ms-ad-config-img13.png)
*Figure 13: Completing the New Zone Wizard.*

* **DNS Forwarders:** In **DNS Manager**, right-click the server name, select <kbd>Properties</kbd>, and go to the <kbd>Forwarders</kbd> tab. 
  * I added public DNS server IPs (e.g., `9.9.9.9` and `1.1.1.1`) here.
* **Client DNS Correction:** I re-checked client devices and the TP-Link router settings to ensure the primary DNS was correctly pointing to the Windows Server's IP (`192.168.30.190`), not the Proxmox host IP.
* **Testing:**
    * On the server, ran `ipconfig /registerdns`to ensure the DC itself was correctly registered.
    * Used `nslookup homelab.lan` on clients.
      * If `nslookup` failed, I ran `nslookup homelab.lan 192.168.30.190` (specifying the server) to determine if the DNS server itself could resolve the name.
    * On clients, ran `ipconfig /flushdns` then `ipconfig /registerdns` to clear out cache and force DNS re-query and re-registrations.
* These sequential steps resolved the DNS issues.

![Successful nslookup test after DNS fixes](/assets/img/posts/ms-ad-config/ms-ad-config-img14.png)
*Figure 14: Successful nslookup indicating DNS is working.*

### 7. Planning and Creating Organizational Units (OUs)

Next, with my OU network design plan structured to manage users, groups, and policies across my simulated departments, I began implementing a hierarchical Active Directory structure. My core strategy for this design, reflecting common industry best practices, focused on achieving a balance between broad security enforcement and granular administrative control. This involved:

* Establishing a domain-level Group Policy Object (GPO) to apply baseline security policies uniformly across all users and computers, ensuring a foundational security posture for the entire network.
* Creating specific Organizational Units (OUs) for different administrative roles (e.g., Server Admins, Department Admins, Tech Support Admins).
* Implementing OU-level GPOs and carefully configured inheritance rules to grant tailored permissions and reverse general restrictions for these designated administrative groups, strictly adhering to the principle of least privilege.
* Strategically blocking inheritance on sensitive OUs, such as those containing server administrator accounts, to protect them from unintended or overly restrictive domain-wide policies.

> Have an [Organizational Unit Design](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/creating-an-organizational-unit-design) in place that meet your immediate and future network needs before creating policies and adding users and devices.
{: .prompt-info}

* To begin creating the design, open **Active Directory Users and Computers (ADUC)** from Server Manager → Tools, or by searching for `dsa.msc`.
* **Create Top-Level OU:** Right-click on the domain (e.g., `homelab.lan`), select <kbd>New</kbd> > <kbd>Organizational Unit</kbd>. Name it (e.g., `Departments`).

![Creating a new OU in ADUC](/assets/img/posts/ms-ad-config/ms-ad-config-img15.png)
*Figure 15: New Object - Organizational Unit dialog box.*

* **Create Sub-OUs:** Right-click on the newly created `Departments` OU and repeat the process to create Sub-OUs for each department (e.g., `Operations`, `Accounting`, `Marketing`, `TechSupport`, `Management`).

![ADUC showing created OU hierarchy](/assets/img/posts/ms-ad-config/ms-ad-config-img16.png)
*Figure 16: Example OU structure within ADUC.*

### 8. Creating New User Accounts

* In **ADUC**, navigate to the appropriate department OU.
* Right-click the OU, select <kbd>New</kbd> → <kbd>User</kbd>.
* Fill in the user details (First name, Last name, User logon name). Use a consistent naming convention.
* Set a password and configure password options (e.g., "User must change password at next logon").

![Creating a new user in ADUC - Step 1](/assets/img/posts/ms-ad-config/ms-ad-config-img17.png)
*Figure 17: New Object - User dialog, first screen.*

![Creating a new user in ADUC - Step 2 (Password)](/assets/img/posts/ms-ad-config/ms-ad-config-img18.png)
*Figure 18: New Object - User dialog, password options.*

* Repeat for all users, placing them in their correct department OUs.

### 9. Joining Client Computers to the Domain

For a Windows 11 Pro client:
* Log into an existing local administrator account on the client PC.
* Open <kbd>Settings</kbd> → <kbd>Accounts</kbd> → <kbd>Access work or school</kbd>.
* Click <kbd>Connect</kbd>.
* At the bottom of the Microsoft account prompt, click the link "<kbd>Join this device to a local Active Directory domain</kbd>".
* Enter your domain name (e.g., `homelab.lan`) → <kbd>Next</kbd>.
* Enter the credentials of a domain user account that has permissions to join computers to the domain (e.g., a domain admin account or a user you've delegated this right to). Follow the prompts.
* Restart the client PC when prompted. You can now log in with domain user credentials.

![Joining Windows 11 to a local Active Directory domain](/assets/img/posts/ms-ad-config/ms-ad-config-img19.png)
*Figure 19: Windows 11 "Set up a work or school account" dialog.*

> Tip: If you encounter issues joining or logging in, ensure the client PC's network adapter is configured to use the AD DNS server (your Windows Server). If problems persist, try joining using domain admin credentials first, then attempt user login.
{: .prompt-tip}

### 10. Implementing GPO Policies and Inheritance Structure Design

Understanding Group Policy inheritance (LSDOU: Local, Site, Domain, OU) is crucial.

* **Design Strategy Review:**
    1.  Create a top-level OU (e.g., `AdminAccounts`) for all administrator accounts to isolate them.
    2.  Create a `DepartmentAdmins` OU and a nested `TechSupportAdmins` OU within it for tiered admin rights.
    3.  Apply broad security GPOs at the domain level (e.g., `Domain Computer Security Policies`).
    4.  Block inheritance on the `AdminAccounts` OU to protect server admins.
    5.  Create specific GPOs at the `DepartmentAdmins` and `TechSupportAdmins` OU levels to grant necessary permissions (least privilege principle) by reversing/overriding certain domain-level restrictions.
* **Creating Admin OUs:** 
  * In ADUC, right-click the domain, select <kbd>New</kbd> → <kbd>Organizational Unit</kbd> to create `AdminAccounts`. Move existing admin users into it. Repeat for `DepartmentAdmins` (under `Departments` OU or at domain level) and `TechSupportAdmins` (under `DepartmentAdmins`).
* **Creating Admin Security Groups:**
    1.  In ADUC, right-click the `DepartmentAdmins` OU, select <kbd>New</kbd> → <kbd>Group</kbd>.
    2.  Name: `DeptAdmins_Grp`, Scope: `Global`, Type: `Security`.
    3.  Add relevant user accounts to the "Members" tab.
    4.  On the "Member Of" tab, add this group to built-in groups like `Administrators` for basic admin rights. For `TechSupportAdmins_Grp`, I also added them to the `Domain Admins` group for higher privileges.

![Adding a group to the built-in Administrators group](/assets/img/posts/ms-ad-config/ms-ad-config-img20.png)
*Figure 20: Making the custom 'Dept Admins Group' a member of the built-in 'Administrators' group.*

> **Real-World Scenario: Employee Promotions**
> In a production environment, when a user is promoted to an administrative role, the standard procedure is to both **move their user object** to the appropriate administrative OU and **add them as a member** to the relevant administrative security groups. The move ensures they receive the correct policies, and the group membership grants them the necessary permissions.
{: .prompt-tip}

### 11. Blocking GPO Inheritance

* Open **Group Policy Management** (from Server Manager → Tools, or `gpmc.msc`).
* Right-click on the `AdminAccounts` OU and select "<kbd>Block Inheritance</kbd>". This prevents GPOs from higher levels (like the domain) from applying to this OU.

### 12. Using Group Policy to Manage & Secure User Accounts

* **Create Domain-Level GPO:** In Group Policy Management, right-click your domain (e.g., `homelab.lan`), and select "<kbd>Create a GPO in this domain, and Link it here...</kbd>". Name it (e.g., `Domain Computer Security Policies`).

![Creating a new GPO linked at the domain level](/assets/img/posts/ms-ad-config/ms-ad-config-img21.png)
*Figure 21: Creating and linking a new GPO.*

* **Edit GPO:** Right-click the new GPO and select <kbd>Edit...</kbd>. Navigate Computer Configuration or User Configuration to set policies.
* **Example Policy (Test):** To test GPO application, I prohibited access to the Control Panel for all users:
    * Path: `User Configuration` > `Policies` > `Administrative Templates` > `Control Panel` > `Prohibit access to Control Panel and PC settings`. Set to **Enabled**.

* **Create OU-Level GPO to Override:** Create another GPO linked to the `DepartmentAdmins` OU. Edit it and set the *same* "Prohibit access to Control Panel..." policy to **Disabled**. This should reverse the domain policy for users in this OU and its children (like `TechSupportAdmins`).

![Editing a GPO to prohibit Control Panel access](/assets/img/posts/ms-ad-config/ms-ad-config-img22.png)
*Figure 22: Enabling the "Prohibit access to Control Panel and PC settings" policy.*

![GPO hierarchy and link order](/assets/img/posts/ms-ad-config/ms-ad-config-img23.png)
*Figure 23: Illustrating GPO linking and (intended) inheritance.*

### 13. Testing GPO Policies

* **Server Admin User:** Logged in as a server admin (in the `AdminAccounts` OU with Block Inheritance). Verified Control Panel access was **not** blocked. Success.
* **Non-Admin User:** Logged in as a regular user. Trying to access Control Panel resulted in a restriction message. Success (domain GPO applied).
    > **Note:** If testing with a user already logged in, run `gpupdate /force` in their command prompt or sign them out and back in to ensure policies apply.
    {: .prompt-tip}
* **Department Admin User:** Logged in as a department admin. Control Panel access was **still blocked**. This was unexpected, as the OU-level GPO should have overridden the domain GPO.

## Troubleshooting & Key Learnings

### 1. GPO Application Failure: The Resolution

* **Observed Problem:** The GPO designed to grant Control Panel access to administrators was not applying. Despite being in the correct security group, administrative users were still blocked by the domain-wide restrictive policy.

* **Diagnostic Process:** I systematically verified the exact location of the administrative user objects (e.g., `Lenovo Testuser1`) within the ADUC hierarchy. By enabling "Advanced Features" in the <kbd>View</kbd> menu of ADUC, I could inspect the `distinguishedName` attribute in each user's properties for definitive proof of their OU path.

* **Key Finding:** The critical discovery was that while the security group (`Dept Admins Group`) was correctly placed in the OU where the "Allow" GPO was linked (`homelab.lan/Admin Accounts/Department Admins`), the **user objects themselves were still located in their original departmental OUs** (e.g., `homelab.lan/Departments/Marketing - Vlan 40`).

* **Core Principle Learned (Crucial for AD Design):** This highlighted a fundamental principle of Group Policy: **GPOs apply primarily based on the Organizational Unit (OU) location of the User or Computer object, not solely on group membership.** Security groups are primarily for granting access permissions to resources (like files and folders), whereas OUs are for policy application and administrative delegation.

* **Action Taken:** I performed a planned move of the administrative user objects from their departmental OUs to the appropriate administrative OUs using the drag-and-drop or "Move" function in ADUC.

* **Verification:** On the client device, after the user objects were moved, I forced a Group Policy update (`gpupdate /force` in an elevated Command Prompt), followed by a sign-out and sign-in of the administrative user.

* **Result: SUCCESS!** The administrative users were now able to access the Control Panel. Correctly placing the user objects within the GPO-linked OU was the solution. The `gpresult /r` command confirmed that both the restrictive domain policy and the overriding administrative policy were being applied correctly.

### 2. Other Lessons Learned During the Project

* **VM Console Interaction (SPICE vs. VNC):**
    * **Problem:** Difficulty with host-to-VM copy/paste despite the QEMU Guest Agent being installed.
    * **Diagnosis:** Realized that while the Proxmox VM's virtual hardware was configured for SPICE, the web-based console was still using VNC.
    * **Lesson Learned:** VM hardware configuration and the host's connection method (console client) must align for full feature utilization. To use SPICE's advanced features like seamless copy/paste, a dedicated SPICE client (like `virt-viewer`) is required on the host machine.

* **Active Directory User Object Property Best Practices:**
    * **Challenge:** When moving administrative users to dedicated OUs, I needed a way to track their original departmental affiliation for organizational clarity.
    * **Solution:** Utilized standard user object properties in ADUC, such as the `Description`, `Department`, and `Office` fields, to record a user's original role or department.
    * **Lesson Learned:** The importance of robust documentation and attribute management within Active Directory for maintaining user context and aiding auditing in complex organizational structures.

