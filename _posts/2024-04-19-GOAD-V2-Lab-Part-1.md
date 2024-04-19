---
layout: post
title: GOAD V2 Lab - Part 1 - Pfsense install
category: [Goad V2, Pfsense, Proxmox]
tags: [homelab, firewall, pfsense, proxmox, goadv2, blueteam]
share: true
---

## Introduction

This guide is based on the guide shared by [mayfly](https://mayfly277.github.io/posts/GOAD-on-proxmox-part1-install/). However, I will not follow the instructions related to the cloud part. I made some modifications accordingly

## Pfsense Web Gui Access from WAN

Don't hesitate to send me your comments.
{:.prompt-warning}

#### Step 1: open a shell 

![Shell](/assets/img/goadv2/pfsense/2024-04-19_10-36.png)

#### Step 2: deactivate packet filter functionality

To deactivate pf in pfSense, run this command 
```bash
pfctl -d
```
![Shell2](/assets/img/goadv2/pfsense/2024-04-18_18-03_1.png)

#### Step 3: Add NAT firewall rule

Navigate to : **Firewall > NAT > Port Forward**

![NAT](/assets/img/goadv2/pfsense/2024-04-18_18-03.png)

The masked NAT IP corresponds to the firewall's LAN interface IP Address
{:.prompt-note}

if you choose **Filter rule association**: *Add associated filter rule*, an associated WAN rule is created in **Firewall > rule > WAN**

![Wan Rule](/assets/img/goadv2/pfsense/2024-04-18_18-02.png)