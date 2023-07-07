---
layout: post
title:  "Proxmox installation guide"
category : Virtualization, Proxmox
tags :  lab, virtualization, proxmox
---

> To be enriched with screenshots for every step.
{: .prompt-warning }

## Step 1: Obtain Proxmox VE 8 ISO

Visit the [Proxmox website](https://www.proxmox.com/) and navigate to the Downloads section.
Download the latest Proxmox VE 8 ISO image suitable for your hardware architecture.

## Step 2: Create a Proxmox VE Installation Media

Burn the downloaded ISO image to a DVD or create a bootable USB drive using software like [Etcher](https://etcher.balena.io/).

## Step 3: Boot from Proxmox VE Installation Media

Insert the installation media into your server or computer.
Configure your system's BIOS/UEFI settings to boot from the installation media.

Restart your system and boot into the Proxmox VE installer.

## Step 4: Proxmox VE Installation

1. Select "Install Proxmox VE" from the boot menu.
2. Choose the appropriate language and keyboard layout for the installation.
3. Accept the End User License Agreement (EULA) to proceed.
4. Select the target disk where Proxmox VE will be installed.
5. Choose the desired installation type (typically "Standard" is recommended).
6. Configure the network settings, including IP address, subnet mask, gateway, and DNS.
7. Set the root password for the Proxmox VE host.
8. Select the timezone for your location.
9. Confirm the installation settings and begin the installation process.

> For more detailed instructions and considerations, you can refer to the official Proxmox VE documentation:
{: .prompt-tip }
```
Proxmox VE Installation Guide: https://pve.proxmox.com/pve-docs/pve-admin-guide/installation/
Proxmox VE Network Configuration: https://pve.proxmox.com/pve-docs/pve-admin-guide/network-configuration.html
Proxmox VE Storage Configuration: https://pve.proxmox.com/pve-docs/pve-admin-guide/storage.html
```

## Step 5: Post-Installation Setup

Once the installation is complete, remove the installation media and reboot the system.

1. Access the Proxmox VE web interface by opening a web browser and entering the IP address of your Proxmox VE host in the address bar.
2. Log in using the root username and the password you set during installation.
3. Follow the official Proxmox VE documentation's instructions to complete the initial configuration, including configuring network settings, storage, and creating a cluster if required.
