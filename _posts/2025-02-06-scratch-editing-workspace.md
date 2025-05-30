---
title: ChatGPT Version Microsoft Server 2022 Install on Proxmox Hypervisor  
date: 2025-05-25  
categories: [Virtualization, Home Lab, Technical-Documentation]  
tags: [proxmox, windows-server, server-2022, virtio, vm, configuration]  
---

## Introduction

After successfully installing the Proxmox hypervisor on a test laptop and confirming network connectivity via the web GUI from another VLAN, I proceeded to install **Microsoft Server 2022** as a VM on Proxmox.

During this process I encountered an access issue: my main PC could not reach the Proxmox GUI while two other devices—one on the same VLAN and one on a different VLAN—could. I ruled out browser conflicts (tested multiple browsers and private/incognito mode) and network issues (`ping`, `tracert`). Ultimately the culprit was Windows Defender Firewall on my PC. I resolved it by creating a firewall rule for the Proxmox management port (8006) and refreshing the network adapter. Detailed troubleshooting steps are in the **Troubleshooting** section.

---

## Initial Setup

1. **Download Windows Server 2022 ISO**  
   - Go to the [Microsoft Evaluation Center](https://www.microsoft.com/evalcenter) and download the 64-bit ISO.  
   - (Optional) Direct ISO: `https://www.microsoft.com/.../Server2022EVAL.iso`

2. **Obtain VirtIO Drivers for KVM**  
   - Download the VirtIO drivers ISO from the [Proxmox VE download page](https://pve.proxmox.com/wiki/Download).  
   - _Tip_: After uploading the Server ISO to Proxmox storage, you can copy the VirtIO download link directly from the Proxmox UI (“Copy Link” → paste in browser) instead of downloading manually.

---

## Configuration Steps

1. **Upload ISOs to Proxmox**  
   1. Access the Proxmox web UI.  
   2. In the left-hand pane, select your storage (e.g., `local`).  
   3. Click the **ISO Images** tab, then **Upload**.  
      - Click <kbd>Select File</kbd> and choose the **Windows Server 2022 ISO**.  
      - _Do not_ alter the auto-generated path.  
      
   ![Upload ISO to Proxmox](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img1.png)  
   *Figure 1: Uploading the Server 2022 ISO.*

2. **Create the VM**  
   1. Click **Create VM** in the top-right of the Proxmox UI.  
   2. **General** tab  
      - **Name**: `MS-Server22`  
      - **Start at boot**: ✔️  
   3. **OS** tab  
      - **Type**: Windows  
      - **Version**: `11/2022/2025`  
      - **CD/DVD**: select your uploaded Server 2022 ISO  
      
      ![Select ISO in VM wizard](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img2.png)  
      *Figure 2: OS tab settings.*  
      
   4. **System** tab  
      - **SCSI Controller**: VirtIO SCSI  
      - **Add TPM**: ✔️  
      
      ![System tab settings](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img3.png)  
      *Figure 3: System tab settings.*  
      
   5. **Disks** tab  
      - Adjust **Disk size** as needed  
      - **Cache**: Write back  
      
      ![Disk settings](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img4.png)  
      *Figure 4: Disk tab settings.*  
      
   6. **CPU** tab  
      - **Type**: host (avoids unsupported socket types)  
      - Configure **Sockets** & **Cores** per your hardware  
      
      ![CPU settings](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img5.png)  
      *Figure 5: CPU tab settings.*  
      
   7. **Memory** tab  
      - Allocate RAM (e.g., 4 GB)  
      - **Ballooning Device**: enabled, min 512 MB  
      
      ![Memory settings](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img6.png)  
      *Figure 6: Memory tab settings.*  
      
   8. **Network** tab  
      - **Model**: VirtIO (best performance)  
      
      ![Network settings](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img7.png)  
      *Figure 7: Network tab settings.*  
      
   9. Click **Finish** to create the VM.

3. **Start the VM and Install**  
   1. In the left pane, right-click your VM → **Start**.  
   2. Click the **>_ Console** button to open the VM display.  
   3. Proceed through the Windows Server installer. When prompted “Where do you want to install?”, you’ll see **No drives found**—this is expected until VirtIO drivers are loaded.  
      
      ![No drives found warning](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img8.png)  
      *Figure 8: “No drives found” warning.*  
      
   4. Click **Load driver**, browse to the VirtIO CD drive:  
      1. `vioscsi\2k22\amd64` → select **Red Hat VirtIO SCSI pass-through controller** → **Next**.  
      2. Repeat for `NetKVM\2k22\amd64` (**Red Hat VirtIO Ethernet Adapter**).  
      3. Repeat for `Balloon\2k22\amd64` (**VirtIO Balloon Driver**).  
      
      ![Load VirtIO drivers](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img9.png)  
      *Figure 9: Loading VirtIO drivers.*  
      
   5. Complete the installation and reboot as prompted.  
   6. When the login screen does not immediately appear, click the left arrow on the console pane, then the **A** button, then the three-square icon to reveal the Windows login.  
      
      ![Access login UI](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img10.png)  
      *Figure 10: Revealing the login screen.*  
      
   7. Log in with your administrator credentials.  
      
      ![Server Manager dashboard](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img11.png)  
      *Figure 11: Server Manager Dashboard.*  

4. **Post-Installation Tasks**  
   1. On the Windows desktop, open **File Explorer**, navigate to the VirtIO CD, and run `virtio-win-gt-x64.msi` to install any remaining drivers.  
      
      ![VirtIO installer](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img12.png)  
      *Figure 12: Running the VirtIO installer.*  
      
   2. Configure a static IP in your chosen VLAN:  
      ```powershell
      New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.40.10 `
        -PrefixLength 24 -DefaultGateway 192.168.40.1
      ```
      
      ![IPv4 properties](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img13.png)  
      *Figure 13: Setting a static IPv4 address.*  
      
   3. Install the QEMU Guest Agent:  
      - From the VirtIO CD, run `qemu-ga-x86_64.msi`.  
      - Verify in Proxmox **Summary** that the VM’s IP is now detected automatically.  
      
      ![Guest Agent summary](/assets/img/posts/ms-server-2022-proxmox/ms-server-2022-proxmox-img14.png)  
      *Figure 14: QEMU Guest Agent confirmation.*  

---

## Troubleshooting

1. **Proxmox GUI Access Blocked by Windows Firewall**  
   - Ensure your PC’s network profile is set to **Private**.  
   - Create an outbound rule in **Windows Defender Firewall**:  
     1. Open **Windows Defender Firewall** → **Outbound Rules** → **New Rule**.  
     2. **Port** → TCP 8006 → **Allow** → apply to all profiles → name: `Proxmox UI Outbound`.  

2. **VM Fails to Start (QEMU exited with code 1)**  
   > **Cause:** Incompatible CPU type or missing VirtIO drivers.  
   {: .prompt-warning}  
   - Updated to the latest VirtIO drivers.  
   - Verified **CPU type** is set to **host**.  
   - Confirmed **SCSI controller** = VirtIO SCSI, **Cache** = Write back, **Balloon** enabled.  

---

## Conclusion

This exercise deepened my understanding of virtual machine deployment and troubleshooting on Proxmox. Key takeaways:

- Proper selection of CPU type and drivers is critical.  
- Windows Defender Firewall can silently block web UI access; always verify firewall rules.  
- Post-installation guest tools (QEMU Agent, VirtIO installer) complete the integration.

Feel free to reach out on [LinkedIn](https://www.linkedin.com/in/jarrod-fueston/) with any questions or feedback!  
