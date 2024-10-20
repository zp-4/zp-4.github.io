---
layout: post
title: Setting Up a Shared Python Development VM
date: 2024-10-20 10:18 +0200
categories: [Proxmox, Virtualization, Development]
tags: [Proxmox, Virtual Machines, Development, Python, RDP]
comments: true
share: true
---

# Comprehensive Guide: Setting Up a Shared Python Development VM (VDI) on Proxmox

This guide provides detailed instructions for setting up a Debian-based Virtual Machine (VM) on Proxmox, specifically tailored for shared Python development. The VM will support multiple users, each with their own development environment, accessible via Remote Desktop Protocol (RDP).

## 1. Create and Configure the VM in Proxmox

1. Log into your Proxmox web interface.
2. Click on "Create VM" and configure as follows:
   - **Name:** PythonDevVM (or any preferred name)
   - **OS:** Choose Debian 11 (or the latest stable version)
   - **CPU:** Allocate at least 4 cores for better performance
   - **RAM:** Allocate at least 16 GB for smooth multi-user operation
   - **Disk:** Allocate at least 100 GB to accommodate multiple users and projects
   - **Network:** Ensure the VM is connected to your network

## 2. Install Debian on the VM

1. Start the VM and boot from the Debian ISO.
2. Follow the Debian installation process:
   - Choose a minimal installation without a desktop environment
   - Set up a root password and create an initial user account

## 3. Initial System Configuration

1. After installation, start the VM and log in as root.
2. Update the system:
   ```bash
   apt update && apt upgrade -y
   ```
3. Install essential tools:
   ```bash
   apt install sudo vim wget curl git build-essential -y
   ```
4. Add your initial user to the sudo group:
   ```bash
   usermod -aG sudo username
   ```

## 4. Install and Configure SSH (Optional but Recommended)

1. Install OpenSSH server:
   ```bash
   apt install openssh-server -y
   ```
2. Configure SSH for key-based authentication (recommended):
   - Edit `/etc/ssh/sshd_config`
   - Set `PasswordAuthentication no`
   - Set `PubkeyAuthentication yes`
3. Restart SSH service:
   ```bash
   systemctl restart ssh
   ```

## 5. Create Additional User Accounts

For each developer who needs access:

1. Create a new user account:
   ```bash
   adduser devuser1
   ```
2. Add the user to necessary groups:
   ```bash
   usermod -aG sudo,ssh devuser1
   ```

Repeat this process for each developer.

## 6. Install Desktop Environment (XFCE)

1. Install XFCE and related packages:
   ```bash
   apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils -y
   ```
2. Install LightDM display manager:
   ```bash
   apt install lightdm -y
   ```

## 7. Set Up Remote Desktop Access (XRDP)

1. Install XRDP:
   ```bash
   apt install xrdp -y
   ```
2. Configure XRDP to use XFCE:
   ```bash
   echo xfce4-session > ~/.xsession
   ```
3. Add all users to the ssl-cert group:
   ```bash
   for user in $(getent passwd | cut -f1 -d: ); do usermod -a -G ssl-cert $user; done
   ```
4. Restart XRDP service:
   ```bash
   systemctl restart xrdp
   ```

## 8. Configure Polkit for Multi-User Sessions

1. Create a Polkit configuration file:
   ```bash
   nano /etc/polkit-1/localauthority/50-local.d/45-allow-colord.pkla
   ```
2. Add the following content:
   ```ini
   [Allow Colord all Users]
   Identity=unix-user:*
   Action=org.freedesktop.color-manager.create-device;org.freedesktop.color-manager.create-profile;org.freedesktop.color-manager.delete-device;org.freedesktop.color-manager.delete-profile;org.freedesktop.color-manager.modify-device;org.freedesktop.color-manager.modify-profile
   ResultAny=yes
   ResultInactive=yes
   ResultActive=yes
   ```

## 9. Install Python and Development Tools

1. Install Python 3 and related tools:
   ```bash
   apt install python3 python3-pip python3-venv python3-dev -y
   ```
2. Install additional Python tools:
   ```bash
   apt install ipython3 python3-doc -y
   ```
3. Verify the installation:
   ```bash
   python3 --version
   pip3 --version
   ```

## 10. Set Up Python Virtual Environment Tools

1. Install virtualenvwrapper:
   ```bash
   pip3 install virtualenvwrapper
   ```
2. Configure virtualenvwrapper for all users:
   ```bash
   echo "export WORKON_HOME=$HOME/.virtualenvs" >> /etc/bash.bashrc
   echo "export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3" >> /etc/bash.bashrc
   echo "source /usr/local/bin/virtualenvwrapper.sh" >> /etc/bash.bashrc
   ```

## 11. Install Development Tools and IDEs

1. Install Visual Studio Code:
   ```bash
   wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
   install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
   echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list
   apt update
   apt install code -y
   ```
2. Install PyCharm Community Edition (optional):
   ```bash
   snap install pycharm-community --classic
   ```

## 12. Install Additional Python Libraries and Tools

1. Install commonly used Python libraries:
   ```bash
   pip3 install numpy pandas matplotlib scikit-learn jupyter
   ```
2. Install database connectors:
   ```bash
   apt install python3-mysqldb python3-psycopg2 -y
   ```

## 13. Configure Git for Each User

For each user, set up Git configuration:

```bash
su - devuser1
git config --global user.name "Developer Name"
git config --global user.email "developer@email.com"
```

## 14. Set Up Shared Project Directories

1. Create a shared project directory:
   ```bash
   mkdir /opt/shared_projects
   chmod 775 /opt/shared_projects
   chown root:developers /opt/shared_projects
   ```
2. Create a 'developers' group and add all dev users to it:
   ```bash
   groupadd developers
   for user in devuser1 devuser2 devuser3; do usermod -aG developers $user; done
   ```

## 15. Configure Firewall

1. Install and configure UFW:
   ```bash
   apt install ufw -y
   ufw allow 22/tcp  # SSH
   ufw allow 3389/tcp  # RDP
   ufw enable
   ```

## 16. Set Up Regular System Updates

1. Create a script for automatic updates:
   ```bash
   nano /usr/local/bin/update-system.sh
   ```
2. Add the following content:
   ```bash
   #!/bin/bash
   apt update
   apt upgrade -y
   apt autoremove -y
   ```
3. Make the script executable:
   ```bash
   chmod +x /usr/local/bin/update-system.sh
   ```
4. Set up a weekly cron job:
   ```bash
   echo "0 2 * * 0 root /usr/local/bin/update-system.sh" > /etc/cron.d/update-system
   ```

## 17. Final Steps and Testing

1. Reboot the VM:
   ```bash
   reboot
   ```
2. Test RDP access for each user from a client machine.
3. Verify that each user can:
   - Log in via RDP
   - Access Visual Studio Code or PyCharm
   - Create and activate Python virtual environments
   - Install Python packages
   - Access shared project directories

## Conclusion

This guide provides a comprehensive setup for a shared Python development VM (VDI) environment on Proxmox. It allows multiple developers to work in isolated environments while sharing resources and project directories. Regular maintenance, security updates, and backups are crucial for keeping this development environment robust and secure.

Remember to customize this setup based on your specific needs, such as adding version control systems, continuous integration tools, or specific Python frameworks relevant to your projects.
