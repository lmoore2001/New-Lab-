# Enterprise Infrastructure Lab

I work in IT support, and I wanted a way to get hands-on with the infrastructure I troubleshoot and support. I took an older Dell OptiPlex, turned it into a **Proxmox VE** virtualization host, and started building out Linux servers and virtual networks. Some things worked right away; others definitely made me troubleshoot. That's part of why I built it.

This repository documents the first two phases: **Proxmox host setup** and **Ubuntu Server with isolated virtual networking**. I included the actual configurations, tests, and problems I ran into, not just a list of technologies.

## Hardware

| Hardware | Role |
| --- | --- |
| Dell OptiPlex 9010, Intel Core i7-3770, 16 GB RAM | Proxmox VE host |
| 476.9 GiB SSD and 232.9 GiB HDD | Local storage |
| NETGEAR unmanaged switch | Ethernet connectivity |
| Windows laptop | Remote administration and testing |

## Phase 1 — Proxmox VE Host Setup

I verified the Dell's CPU, memory, storage, and **Intel VT-x** support, then installed **Proxmox VE 9.2** on the SSD. I configured a static management IP (`10.0.0.234/24`), gateway, DNS, and a DHCP reservation on the router so the management address wouldn't conflict with another device.

I also had to work through a **PXE-E53 boot error**, an oddly reported USB capacity, and Proxmox updates failing against subscription-only enterprise repositories. After switching to the **pve-no-subscription** repository, I updated the host, rebooted, and verified the dashboard from my Windows laptop.

**Skills:** virtualization, hardware assessment, UEFI/PXE troubleshooting, static IP addressing, DHCP reservations, Linux package management, remote administration.

[Phase 1 — installation, troubleshooting, commands, and screenshots](../../issues/1)

## Phase 2 — Ubuntu Server & Virtual Networking

Next I installed **Ubuntu Server 24.04.5 LTS** on **VM 100** (2 vCPU, 2 GiB RAM, 25 GiB disk) with a VirtIO NIC on `vmbr0`. I verified DHCP, DNS resolution, the default route, internet connectivity, **SSH**, and **QEMU Guest Agent**. I also used **Tailscale** to SSH into the Ubuntu VM from my phone over cellular data, without opening an SSH port on my router.

Then I created an isolated Proxmox Linux bridge, **`vmbr1`**, for two Ubuntu test VMs.

| VM | IPv4 | Network |
| --- | --- | --- |
| VM 100 — `ubuntu-server-01` | `10.0.0.160/24` (last observed DHCP lease) | `vmbr0` home LAN |
| VM 101 — `ubuntu-lab-01` | `192.168.100.10/24` | `vmbr1` isolated |
| VM 102 — `ubuntu-lab-02` | `192.168.100.11/24` | `vmbr1` isolated |

```text
Home router 10.0.0.1
      |
  nic0 / vmbr0 — Proxmox 10.0.0.234
      |
  VM 100 Ubuntu 10.0.0.160
      └── Tailscale → cellular SSH verified

  vmbr1 — isolated 192.168.100.0/24
      |                        |
  VM 101 .10                VM 102 .11
      └──── ping passed ───────┘
  VM 102 → home router: ping failed (expected)
```

Both isolated VMs could ping each other. VM 102 had no default route, and its ping to the home gateway failed as expected. That's a useful first check of **network segmentation**, though a failed ping alone isn't a complete security test. The core setup is working; I'm still organizing screenshots and completing final bridge/NIC verification.

**Skills:** Linux administration, Proxmox VMs, VirtIO, Linux bridges, TCP/IP, DHCP, DNS, SSH, Tailscale, routing, network segmentation, connectivity troubleshooting.

[Phase 2 — setup, network tests, and remaining verification](../../issues/2)

## What I took away

Getting a VM running is one thing. Being able to check its routes, explain which network it's on, troubleshoot access, and prove a change worked is what I wanted to practice. That's been the most useful part of the lab so far.
