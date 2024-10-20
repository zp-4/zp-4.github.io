---
layout: post
title: Migrating Your Proxmox Virtual Machines
categories: [Proxmox, Virtualization, Migration]
tags: [Proxmox, Virtual Machines, Backup, Storage]
date: 2024-10-20 09:29 +0200
---

# Comprehensive Guide to Migrating Proxmox VMs to New Disks

Migrating a Proxmox server to new disks can seem daunting, especially when preserving your valuable virtual machines (VMs) and containers (CTs). This comprehensive guide will walk you through each step to ensure a smooth and data-safe migration, whether you're retaining the existing storage configuration or creating a new one.

## 1. Backing Up Your Virtual Machines and Containers: A Crucial Step

Before making any changes, the first essential step is to back up your VMs and CTs. This precaution ensures that you can safely restore your environments no matter what happens.

### Backing Up Using the Proxmox Web Interface

1. Access the Proxmox web interface.
2. Navigate to "Datacenter" > "Backup".
3. Select the VMs/CTs to back up and the backup location (this could be an external disk, a NAS, or any other available storage).
4. Start the backup and wait for it to complete. This process can take some time depending on the size of your VMs/CTs.

### Backing Up Using Terminal Commands

You can create either snapshot backups (which are typically faster but require more storage space) or full backups (which are slower but more space-efficient). Here's how to do both:

To back up a single VM/CT:

```bash
# For a VM snapshot backup (replace 100 with your VM ID)
vzdump 100 --mode snapshot --compress zstd --storage local

# For a VM full backup
vzdump 100 --mode stop --compress zstd --storage local

# For a CT snapshot backup (replace 101 with your CT ID)
vzdump 101 --mode snapshot --compress zstd --storage local

# For a CT full backup
vzdump 101 --mode stop --compress zstd --storage local
```

To back up all VMs and CTs at once:

```bash
# Snapshot backup of all VMs and CTs
vzdump --all --mode snapshot --compress zstd --storage local

# Full backup of all VMs and CTs
vzdump --all --mode stop --compress zstd --storage local
```

These commands will create backups in the specified storage (in this case, 'local'). You can change 'local' to any configured storage location.

### Exporting Configuration Files

In addition to full backups, export the configuration files of the VMs/CTs:

```bash
# For VMs
cp /etc/pve/qemu-server/*.conf /path/to/backup/location/

# For CTs
cp /etc/pve/lxc/*.conf /path/to/backup/location/
```

These `.conf` files contain critical details about each VM/CT, such as attached disks and allocated resources.

## 2. Optional: Installing New Hardware and Reinstalling Proxmox

If you're upgrading your hardware, follow these steps:

1. Shut down your Proxmox server.
2. Install the new disks in your server.
3. Boot from your Proxmox installation media.
4. Perform a fresh installation of Proxmox on your chosen disk.
5. Configure your network settings as needed.

If you're not changing hardware, you can skip this step and proceed to restoring your VMs and CTs.

## 3. Restoring Your Virtual Machines and Containers: Two Approaches

### Option 1: Retain the Previous Storage Configuration

If you've reconfigured your storage similarly to the previous setup (i.e., keeping the same pool and volume names), the restoration process will be straightforward:

#### Using the Web Interface

1. Access "Datacenter" > "Backup" in Proxmox.
2. Select "Restore" for each VM/CT.
3. Choose the appropriate backup file and restore location.

#### Using Terminal Commands

To restore a single VM/CT:

```bash
# Replace 'vzdump-qemu-100-2023_10_20-10_00_00.vma.zst' with your actual backup file name
# Replace '100' with the VM/CT ID you want to restore to (can be different from the original)
qmrestore /var/lib/vz/dump/vzdump-qemu-100-2023_10_20-10_00_00.vma.zst 100
```

### Option 2: Change the Storage Configuration

If you've modified your storage configuration (new pool names, new volumes, etc.), follow these steps:

1. **Restore via the Proxmox interface:**
   - Go to "Datacenter" > "Backup" and select "Restore".
   - During restoration, specify the new storage location for each VM/CT.

2. **Manual restore using terminal commands:**
   When restoring to a different storage location, you can specify the new location using the `--storage` option:

   ```bash
   qmrestore /var/lib/vz/dump/vzdump-qemu-100-2023_10_20-10_00_00.vma.zst 100 --storage new-storage-pool
   ```

   This command restores the VM with ID 100 to the storage pool named 'new-storage-pool'.

   If you need to specify different locations for different disks, you can use the `--target` option:

   ```bash
   qmrestore /var/lib/vz/dump/vzdump-qemu-100-2023_10_20-10_00_00.vma.zst 100 \
     --target new-storage-pool:vm-100-disk-0 \
     --target other-storage-pool:vm-100-disk-1
   ```

   This command restores the VM with ID 100, placing its first disk (disk-0) on 'new-storage-pool' and its second disk (disk-1) on 'other-storage-pool'.

## 4. Finalization and Testing

After restoring or importing your VMs/CTs:

1. Start each VM/CT and verify it boots correctly.
2. Check that all services within the VM/CT are running as expected.
3. Test network connectivity and performance.
4. Verify that all data is intact and accessible.
5. Monitor resource usage to ensure it aligns with pre-migration levels.

## Bonus: Automating Backups and Restores

To streamline your backup and restore processes, *and if you are not comfortable with the Proxmox web interface*, consider implementing automation scripts. Here's a basic example of a backup automation script:

```bash
#!/bin/bash

# Set variables
BACKUP_DIR="/path/to/backup/location"
LOG_FILE="/var/log/proxmox_backup.log"
EMAIL="your@email.com"

# Function to send email
send_email() {
    echo "$1" | mail -s "Proxmox Backup Status" $EMAIL
}

# Start backup process
echo "Starting backup process at $(date)" >> $LOG_FILE

# Backup all VMs and CTs
vzdump --all --compress zstd --mode snapshot --storage $BACKUP_DIR

# Check if backup was successful
if [ $? -eq 0 ]; then
    echo "Backup completed successfully at $(date)" >> $LOG_FILE
    send_email "Proxmox backup completed successfully"
else
    echo "Backup failed at $(date)" >> $LOG_FILE
    send_email "Proxmox backup failed"
fi

# Clean up old backups (keeping last 7 days)
find $BACKUP_DIR -type f -mtime +7 -name '*.vma.zst' -delete

echo "Backup process finished at $(date)" >> $LOG_FILE
```

To use this script:

1. Save it as `proxmox_backup.sh` in a suitable location (e.g., `/root/scripts/`).
2. Make it executable: `chmod +x /root/scripts/proxmox_backup.sh`
3. Set up a cron job to run it regularly:
   ```
   0 1 * * * /root/scripts/proxmox_backup.sh
   ```
   This will run the backup every day at 1 AM.

Remember to adjust the variables at the top of the script to match your environment.

For restore automation, you would need a more complex script that can handle different scenarios (e.g., single VM/CT restore, full system restore). Such a script would need to be carefully designed and tested to ensure it doesn't accidentally overwrite or delete important data.
Here's an example of a script that restores all VMs and CTs from a specified backup directory:

```bash
#!/bin/bash

# Set variables
BACKUP_DIR="/path/to/backup/location"
RESTORE_STORAGE="your-storage-pool"
LOG_FILE="/var/log/proxmox_restore_all.log"
EMAIL="your@email.com"

# Function to send email
send_email() {
    echo "$1" | mail -s "Proxmox Restore All Status" $EMAIL
}

# Function to restore a single VM or CT
restore_single() {
    local backup_file="$1"
    local vm_id=$(basename "$backup_file" | grep -oP '(?<=vzdump-qemu-)\d+(?=-)')
    
    echo "Restoring VM/CT $vm_id from $backup_file" >> $LOG_FILE
    
    if qmrestore "$backup_file" "$vm_id" --storage "$RESTORE_STORAGE"; then
        echo "Restore of VM/CT $vm_id completed successfully" >> $LOG_FILE
    else
        echo "Restore of VM/CT $vm_id failed" >> $LOG_FILE
        return 1
    fi
}

# Main restore process
echo "Starting restore process for all VMs and CTs at $(date)" >> $LOG_FILE

# Find all backup files
backup_files=$(find "$BACKUP_DIR" -name "vzdump-*.vma.zst" -type f)

if [ -z "$backup_files" ]; then
    echo "No backup files found in $BACKUP_DIR" >> $LOG_FILE
    send_email "Proxmox restore all failed: No backup files found"
    exit 1
fi

# Counter for successful and failed restores
success_count=0
fail_count=0

# Iterate through each backup file and restore
for backup_file in $backup_files; do
    if restore_single "$backup_file"; then
        ((success_count++))
    else
        ((fail_count++))
    fi
done

# Log results
echo "Restore process completed at $(date)" >> $LOG_FILE
echo "Successfully restored: $success_count" >> $LOG_FILE
echo "Failed to restore: $fail_count" >> $LOG_FILE

# Send email notification
if [ $fail_count -eq 0 ]; then
    send_email "Proxmox restore all completed successfully. Total restored: $success_count"
else
    send_email "Proxmox restore all completed with some failures. Successful: $success_count, Failed: $fail_count"
fi
```

This restore script automates the process of restoring all VMs and CTs from a specified backup directory. Here's how to use it:

1. Save the script as `proxmox_restore_all.sh` in a suitable location (e.g., `/root/scripts/`).
2. Make it executable: `chmod +x /root/scripts/proxmox_restore_all.sh`
3. Edit the script to set the correct values for `BACKUP_DIR`, `RESTORE_STORAGE`, and `EMAIL`.
4. Run the script:
   ```
   ./proxmox_restore_all.sh
   ```

This script performs the following actions:

1. It searches for all backup files (with the `.vma.zst` extension) in the specified backup directory.
2. For each backup file found, it extracts the VM/CT ID from the filename and attempts to restore it.
3. It keeps track of successful and failed restores.
4. After attempting to restore all VMs and CTs, it logs the results and sends an email notification.

The script restores all VMs and CTs to the storage pool specified in the `RESTORE_STORAGE` variable. You can modify this to use different storage pools for different types of VMs/CTs if needed.


## Conclusion

Migrating your Proxmox virtual machines and containers to new hardware doesn't have to be a daunting task. With proper preparation, including comprehensive backups and careful planning of the restoration process, you can modernize your infrastructure while keeping your valuable data safe. By following these steps and considering automation for routine tasks, you can ensure a smooth migration process and improve your overall system management.

Feel free to share this guide with your colleagues or on your networks. Happy migrating!
