---
title: Scratch Active Directory Configuration Guide
date: 2025-05-30
excerpt: "Step-by-step guide to install and configure Active Directory Domain Services, DNS, OUs, user accounts, Group Policies, and file permissions in a home lab environment, with troubleshooting tips."
categories: [Active Directory, Home Lab, Windows Server, Academic-Project]
tags: [active-directory, ad-ds, dns, gpo, ou, windows-server]
last_modified_at: 2025-05-30
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

## Prerequisites & Planning

1. **DNS readiness:**  
   Ensure clients can resolve the domain controller’s hostname, either by setting the server as primary DNS or configuring DNS forwarding.

> **Tip:** Plan for role reversal when the server trial expires—document DNS/DHCP settings and back up AD DS data via `<kbd>dcpromo</kbd>` before demotion.  
{: .prompt-tip}

2. **Impact assessment:**  
   - Identify server roles (DNS, DHCP, AD DS) to revert post-trial.  
   - Plan data migration off the server before expiration.

---

## 1. Assign the Server a Name

1. In **Server Manager**, click <kbd>Local Server</kbd>.  
2. Enter a descriptive name (e.g., `HOMELAB-DC1`) and click **OK**.  
3. Restart when prompted.  

![Server Name Assignment](/assets/img/posts/ms-ad-config/ms-ad-config-img1.png)  
*Figure 1: Assigning a descriptive server name.*

![Restart Prompt](/assets/img/posts/ms-ad-config/ms-ad-config-img2.png)  
*Figure 2: Restart to apply server name change.*

---

## 2. Install AD DS and DNS Roles

1. In **Server Manager**, click <kbd>Manage</kbd> → <kbd>Add Roles and Features</kbd>.  
2. Choose **Role-based or feature-based installation**, select your server.  
3. Under **Server Roles**, check:  
   1. **Active Directory Domain Services** → click **Add Features**.  
   2. **Active Directory Domain Services** → click **Add Features**.
4. Proceed through the wizard and click **Install**.  
5. Close the wizard when complete.  

![Add Roles Wizard](/assets/img/posts/ms-ad-config/ms-ad-config-img3.png)  
*Figure 3: Selecting AD DS and DNS roles.*

---

## 3. Promote to Domain Controller

1. In **Server Manager**, click the flag icon → **Promote this server to a domain controller**.  
2. Select **Add a new forest**, enter root domain `homelab.lan`.  
3. Set a strong DSRM password and ignore any DNS delegation warning.  

> **Warning:** In production, configure DNS delegation; for home lab it’s safe to proceed.  
{: .prompt-warning}

![AD DS Wizard – Deployment Configuration](/assets/img/posts/ms-ad-config/ms-ad-config-img4.png)  
*Figure 4: Deployment options in AD DS Configuration Wizard.*

---

## 4. Configure DNS Forwarding (DNS Proxy)

You can either set your server as primary DNS or forward queries:

- **Server DNS:** Clients point to `192.168.30.190`; risk of downtime if server offline.  
- **Router DNS forwarding:** In your router GUI under VLAN DNS settings, forward to server (`192.168.30.190`) and fallback to Quad9 (`9.9.9.9`).  

![Router DNS Forwarding](/assets/img/posts/ms-ad-config/ms-ad-config-img5.png)  
*Figure 5: Configuring DNS forwarding in TP-Link GUI.*

---

## 5. Verify DNS Settings on Server

1. Open **DNS Manager** → **Forward Lookup Zones** → confirm `homelab.lan`.  
2. Create a **Reverse Lookup Zone** for `192.168.30.x`.  
3. Add forwarders (`9.9.9.9`, `1.1.1.1`).  
4. On a client, run:
   ```bash
   nslookup homelab.lan
   ipconfig /registerdns
   ipconfig /flushdns

## 6. Plan & Create Organizational Units (OUs)

1. In **Active Directory Users and Computers** (ADUC), right-click the domain (`homelab.lan`) → **New** → **Organizational Unit**.  
2. Create a top-level OU named **Departments**.  
3. Under **Departments**, create sub-OUs for each department (Operations, Accounting, Marketing, Tech Support).

![Creating Departments OU](/assets/img/posts/ms-ad-config/ms-ad-config-img11.png)  
*Figure 11: Creating the top-level “Departments” OU.*

![Creating Sub-OUs](/assets/img/posts/ms-ad-config/ms-ad-config-img12.png)  
*Figure 12: Adding sub-OUs under “Departments.”*

---

## 7. Create User Accounts

1. In ADUC, navigate to each department OU → **New** → **User**.  
2. Enter user details (username, password), configure account options, then click **Next** → **Finish**.  
3. Repeat for all users, ensuring they’re placed in the correct OU.

![New User Wizard](/assets/img/posts/ms-ad-config/ms-ad-config-img13.png)  
*Figure 13: Filling out the New User Wizard.*

![User List in OU](/assets/img/posts/ms-ad-config/ms-ad-config-img14.png)  
*Figure 14: User accounts visible in their department OU.*

---

## 8. Join Clients to the Domain

1. On a Windows 11 client: **Settings** → **Accounts** → **Access work or school** → **Connect**.  
2. Click **Join this device to a local Active Directory domain**, enter `homelab.lan`, then provide domain credentials.  
3. Restart the client to apply group membership.

> **Tip:** Verify the client’s DNS points to `192.168.30.190` before joining to avoid name-resolution failures.  
{: .prompt-tip}

---

## 9. Design & Apply Group Policies

1. Open **Group Policy Management** (GPMC).  
2. Right-click `homelab.lan` → **Create a GPO in this domain** → Name it **Domain Computer Security Policies**.  
3. Link it at the domain root to apply baseline security settings.  
4. In GPMC, right-click the **Server Admins** OU → **Block Inheritance** to protect critical accounts.

![GPMC – New GPO](/assets/img/posts/ms-ad-config/ms-ad-config-img15.png)  
*Figure 15: Creating and linking a domain-level GPO.*

![Blocking Inheritance](/assets/img/posts/ms-ad-config/ms-ad-config-img16.png)  
*Figure 16: Blocking inheritance on the Server Admins OU.*

---

## 10. Test GPO Enforcement

1. On a non-admin client, run:
    ```powershell
    gpupdate /force
    ```
2. Verify that prohibited settings (e.g., Control Panel access) are blocked.  
3. On a Dept Admin account, confirm those settings are reversed under their OU’s GPO.

> **Warning:** Always use `Resultant Set of Policy (RSoP)` or `gpresult /h report.html` to diagnose unexpected GPO behavior.  
{: .prompt-warning}

---

## 11. Configure File & Folder Permissions

1. On a file server share, right-click the folder → **Properties** → **Security** → **Advanced** → **Disable inheritance** → **Convert inherited permissions to explicit**.  
2. Remove unwanted groups, add **Tech Support Admins** with **Full Control**, and **Marketing Users** with **Read** only.  

![Folder Permissions](/assets/img/posts/ms-ad-config/ms-ad-config-img17.png)  
*Figure 17: Securing folder permissions via Advanced Security Settings.*

---

## 12. Verify Access Scenarios

1. **Tech Support Admin:** Should have full control—test by creating, modifying, and deleting files.  
2. **Regular User:** Should only read—attempting write/delete operations should fail with an “Access Denied” message.  

> **Tip:** Use `icacls \\server\share` on a client to quickly audit NTFS permissions.  
{: .prompt-tip}

---

## 13. Next Steps & Future Work

- **Advanced NTFS scenarios:** Explore nested permissions and auditing.  
- **GPO inheritance deep dive:** Document use of security filtering and loopback processing.  
- **Backup & recovery:** Implement System State backups and AD DS snapshots.

---

## Conclusion

Setting up AD DS in my home lab has reinforced my skills in domain services, DNS, OU design, GPO management, and security planning. The hands-on troubleshooting—especially around inheritance and permissions—demonstrates my methodical approach to problem-solving and my ability to document complex processes clearly for both technical peers and hiring managers alike.


<!-- You can also embed raw HTML for additional callouts if needed -->
<blockquote class="bs-callout bs-callout-warning">
  <h4>Note:</h4>
  <p>Be sure to test DNS resolution on all VLANs after changing forwarding settings.</p>
</blockquote>
