---
title: "Installing Linux on a Windows PC using Virtualisation (Hypervisor Guide)"
datePublished: Sun Aug 24 2025 19:24:54 GMT+0000 (Coordinated Universal Time)
cuid: cmeq2v4if000002jv7qv5hg9r
slug: installing-linux-on-a-windows-pc-using-virtualisation-hypervisor-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1756060937863/bb9cc20b-ef23-413d-919d-fbab279e1c1f.jpeg
tags: linux-for-beginners, linux-basics

---

Linux is one of the most powerful and flexible operating systems available today. Many developers, system administrators, and cloud enthusiasts prefer Linux for its open-source nature, security, and compatibility with servers.

But what if you are a **Windows user** and want to explore Linux without removing or dual-booting your existing OS? The answer lies in **Virtualisation** – using a **Hypervisor** to run Linux inside Windows like a separate computer.

In this article, I’ll guide you step by step on how to install Linux on your Windows PC using a hypervisor such as **VirtualBox** or **VMware Workstation Player**.

---

## 🔑 What is Virtualisation?

**Virtualisation** is a technology that allows you to create and run multiple operating systems (called **Virtual Machines**) on a single physical computer.

A **Hypervisor** (Virtual Machine Manager) is the software that enables virtualisation by managing these virtual machines.

Popular hypervisors include:

* **Oracle VirtualBox** (Free & Open Source)
    
* **VMware Workstation Player** (Free for personal use)
    
* **Hyper-V** (Built into Windows Pro/Enterprise editions)
    

---

## 💻 Why Install Linux on Windows via Virtualisation?

* ✅ No risk of losing Windows data (unlike dual boot)
    
* ✅ Run Linux and Windows side by side
    
* ✅ Great for learning Linux commands, scripting, and server setup
    
* ✅ Easy to delete or reset Linux if needed
    

---

## 🛠️ Prerequisites

Before we start, make sure you have:

1. **Windows PC** (with at least 8GB RAM recommended)
    
2. **Hypervisor software** (VirtualBox or VMware)
    
3. **Linux ISO file** (Ubuntu, Fedora, Debian, etc. – download from official website)
    
4. **Enough disk space** (20GB+ free storage)
    

---

## ⚡ Step-by-Step Installation Guide

### **Step 1: Enable Virtualization in BIOS**

1. Restart your PC and enter BIOS/UEFI (press `F2`, `Del`, or `Esc` depending on your manufacturer).
    
2. Locate **Intel VT-x** or **AMD-V** option and enable it.
    
3. Save and exit BIOS.
    

---

### **Step 2: Install a Hypervisor**

* Download and install **VirtualBox** from [virtualbox.org](https://www.virtualbox.org/)  
    OR
    
* Install **VMware Workstation Player** from [vmware.com](https://www.vmware.com/products/workstation-player.html)
    

---

### **Step 3: Create a New Virtual Machine**

1. Open VirtualBox/VMware.
    
2. Click **New** → Enter a name (e.g., “Ubuntu VM”).
    
3. Select **Linux** as the type and choose the version (e.g., Ubuntu 64-bit).
    
4. Allocate memory (at least **2GB**, preferably 4GB+).
    
5. Create a **virtual hard disk** (20GB+ recommended).
    

---

### **Step 4: Mount the Linux ISO**

1. Select your newly created VM.
    
2. Go to **Settings → Storage**.
    
3. Add the downloaded Linux ISO file as a **virtual CD/DVD drive**.
    

---

### **Step 5: Boot and Install Linux**

1. Start the Virtual Machine.
    
2. It will boot into the Linux installer.
    
3. Choose **Install Linux**.
    
4. Follow the on-screen steps:
    
    * Select Language & Keyboard layout
        
    * Set up Username & Password
        
    * Allocate storage (use default guided installation)
        
5. Wait for installation to complete.
    

---

### **Step 6: Start Using Linux**

* After installation, restart the VM.
    
* Log in with your username and password.
    
* You now have a fully functional Linux system running inside Windows! 🎉
    

---

## ⚙️ Tips for Better Performance

* Install **Guest Additions (VirtualBox)** or **VMware Tools** to enable better graphics, copy-paste, and file sharing.
    
* Allocate more RAM and CPU cores if your PC allows.
    
* Use **Shared Folders** for easy file transfer between Windows and Linux.
    

---

## 🎯 Conclusion

Installing Linux on Windows through virtualization is the **safest and easiest way** to get hands-on experience with Linux. Whether you’re a beginner exploring commands or a cloud enthusiast practicing server setups, a **Linux VM** is the perfect playground.

👉 Next step: Try experimenting with Linux commands, setting up servers, or even simulating cloud environments directly on your PC!

---