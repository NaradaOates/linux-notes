# Running Windows 11 on Unsupported Hardware via KVM/QEMU Virtualisation

## Motivation Behind Project
There are two reason for why I chose this as a project. First, my reason for deciding to install Ubuntu was becasue my laptop did not meet Microsoft's official requirements for the Windows 11 upgrade. I wanted to confirm that virtualisation could circumvent this limitation and get Windows 11 running on my hardware regardless. Second, I wanted to experience virtualisation first hand; setting up and configuring a VM from scratch on Linux is a practical skill, and this was an ideal opportunity to put it into practice.
 
## Overview
This project documents the process of running Windows 11 on an Intel i5-7200U laptop with 8GB DDR4 RAM - hardware that does not meet Microsoft's official Windows 11 upgrade requirements. The goal was to prove that Windows 11 could run on unsupported hardware using KVM/QEMU virtualisation on Ubuntu.
 
**Host System:** Ubuntu (Intel i5-7200U, 8GB DDR4)  
**Virtualisation Stack:** QEMU/KVM + Virt-Manager  
**Guest OS:** Windows 11 Pro  
**VM Resources:** 4GB RAM, 2 vCPUs, 80GB disk (HDD)

---
 
## Why This Works
Microsoft enforces hardware requirements (TPM 2.0, Secure Boot, 4GB RAM minimum) at the OS level and not the hardware level. By using KVM/QEMU to emulate compliant hardware, these checks can be satisfied or bypassed entirely, even on machines that would otherwise be ineligible for the upgrade.
 
---
 
## Process
 
### 1. Environment Setup
Installed the KVM/QEMU stack and Virt-Manager on Ubuntu:
 
```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients \
  bridge-utils virt-manager ovmf
```
 
Added the user to the required groups:
```bash
sudo adduser $USER kvm
sudo adduser $USER libvirt
```
 
### 2. ISO Preparation
- Downloaded the **Windows 11 ISO** from Microsoft's official site
- Downloaded the **VirtIO drivers ISO** from the official Fedora repository for paravirtualised disk and network performance
### 3. VM Configuration
Created a new VM in Virt-Manager with the following key settings:
 
| Setting | Value |
|---|---|
| Firmware | UEFI x86_64 (OVMF_CODE_4M.fd) |
| TPM | Emulated, CRB, v2.0 |
| Disk bus | VirtIO |
| Network model | VirtIO |
| Storage | 80GB qcow2 on 1TB HDD (/mnt/storage) |
 
The VM was deliberately placed on the HDD rather than the SSD to preserve Ubuntu's primary drive.
 
---
 
## Troubleshooting
 
### UEFI Shell on Boot
The VM dropped into a UEFI interactive shell instead of booting the Windows ISO automatically. Resolved by manually invoking the bootloader from the shell:
 
```
fs0:
EFI\BOOT\BOOTX64.EFI
```
 
**Note:** The VM was defaulting to a US keyboard layout, meaning the `\` character was mapped to the `#` key. This was a UK/US layout mismatch that required identifying before the command could be entered.
 
### Windows 11 Setup - Secure Boot Requirement
Windows 11 setup blocked installation citing a lack of Secure Boot support. Bypassed using the registry `LabConfig` method via Command Prompt (`Shift + F10`):
 
```
regedit
```
 
Under `HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig`, created the following DWORD (32-bit) values:
 
| Value | Data |
|---|---|
| BypassTPMCheck | 1 |
| BypassSecureBootCheck | 1 |
| BypassRAMCheck | 1 |
 
### No Drives Visible During Installation
Windows could not detect the virtual disk due to missing VirtIO storage drivers. Resolved by loading the driver manually during setup:
 
**Path on VirtIO ISO:** `viostor\w11\amd64`
 
### No Internet After Installation
The VirtIO network adapter was listed as an unknown "Ethernet Controller" in Device Manager. Resolved by manually pointing Windows to the VirtIO network driver:
 
**Path on VirtIO ISO:** `NetKVM\w11\amd64`
 
Additional unknown devices (`PCI Device`, `PCI Simple Communications Controller`) were resolved via:
- `Balloon\w11\amd64`
- `vioserial\w11\amd64`
---
 
## Result
Windows 11 Pro is fully operational on hardware that does not meet Microsoft's official system requirements. The VM boots reliably, connects to the internet, and performs well within the constraints of the shared hardware resources.
 
---
 
## Key Takeaways
- KVM/QEMU with OVMF provides a convincing enough hardware environment to satisfy Windows 11's requirements at the firmware level
- The `LabConfig` registry bypass is applicable to both virtual and physical unsupported hardware installations
- VirtIO drivers are essential — without them, Windows cannot detect the virtual disk or network adapter at install time
- Resource allocation matters on an 8GB system: a 50/50 split between host and guest provides a stable balance
