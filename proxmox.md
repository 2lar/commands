# Homelab Storage Architecture: Dell Laptop NFS Node

**Date:** December 12, 2025  
**Role:** Storage Node (NFS)  
**Hardware:** Dell Inspiron 5570 (i7-8550u / 1TB SSD)  
**OS:** Ubuntu Desktop (Existing)

## 1. Overview
This setup transforms the Dell Laptop into a Network Attached Storage (NFS) node for a Proxmox Cluster. The goal is a distributed system where:
* **HP Elite (Proxmox):** Handles all Compute (VMs, Containers, Logic).
* **Dell Laptop (Ubuntu):** Handles all Bulk Storage (ISOs, Backups, Large Datasets).

This configuration allows the Dell to remain a usable Ubuntu workstation while serving storage in the background.

---

## 2. Prerequisites
* **Static IP on Dell:** Ensure the Dell has a fixed IP (e.g., `10.0.0.15`).
* **Static IP on HP Server:** Required for secure firewall rules (e.g., `10.0.0.20`).
* **Lid Settings:** Ensure Dell does not sleep when lid is closed (`/etc/systemd/logind.conf` -> `HandleLidSwitch=ignore`).

---

## 3. Server-Side Setup (The Dell)

### Step A: Install NFS Kernel Server
Install the necessary packages to broadcast the file system.
```bash
sudo apt update
sudo apt install nfs-kernel-server
````

### Step B: Prepare the Directory

Create the folder that will be shared with the network.

```bash
# Create directory (Change 'eser' to your actual username)
mkdir -p /home/eser/proxmox_share

# Verify path
pwd
# Output should be: /home/eser/proxmox_share
```

### Step C: Configure Exports

Edit the exports file to define *what* to share and *who* can access it.

```bash
sudo nano /etc/exports
```

Add the following line to the bottom of the file.

> **Security Note:** Replace `<HP_SERVER_IP>` with the specific IP of the Proxmox server (e.g., `10.0.0.20`) to prevent unauthorized access from other devices on the network.

```text
/home/eser/proxmox_share <HP_SERVER_IP>(rw,sync,no_subtree_check,no_root_squash)
```

  * `rw`: Read/Write access.
  * `sync`: Writes are confirmed before continuing (safer for data).
  * `no_root_squash`: Allows Proxmox root user to write files (critical for backups).

### Step D: Configure Firewall (UFW)

Ubuntu blocks incoming connections by default. Allow traffic specifically from the Proxmox server.

```bash
# Allow NFS traffic from the HP Server only
sudo ufw allow from <HP_SERVER_IP> to any port nfs

# Reload firewall to apply changes
sudo ufw reload

# Verify status
sudo ufw status
```

### Step E: Apply and Verify

Restart the NFS service to publish the share.

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server

# Verification: Check if the folder is being exported
sudo exportfs -v
```

-----

## 4\. Client-Side Setup (HP Proxmox)

1.  Log into Proxmox Web Interface (`https://<HP_IP>:8006`).
2.  Navigate to **Datacenter** \> **Storage** \> **Add** \> **NFS**.
3.  Enter the Configuration:
      * **ID:** `Dell-Storage` (or `Dell-Laptop`)
      * **Server:** `<DELL_IP_ADDRESS>` (e.g., `10.0.0.15`)
      * **Export:** Select `/home/eser/proxmox_share` (Should auto-populate).
      * **Content:** Select All (Disk Image, ISO, Backup, Container, Snippets).
4.  Click **Add**.

*The storage should now appear in the left sidebar.*

-----

## 5\. VM Setup (Shared Storage inside VMs)

To give a specific VM (e.g., Ubuntu Server) direct access to the Dell storage.

### Step A: Install Client Tools (Inside VM)

```bash
sudo apt update && sudo apt install nfs-common
```

### Step B: Manual Mount (Temporary Test)

```bash
mkdir ~/dell_data
sudo mount <DELL_IP>:/home/eser/proxmox_share ~/dell_data
```

### Step C: Auto-Mount on Boot (Permanent)

Edit the filesystem table.

```bash
sudo nano /etc/fstab
```

Add this line at the bottom:

```text
<DELL_IP>:/home/eser/proxmox_share  /home/vm_user/dell_data  nfs  defaults  0  0
```

Test the configuration:

```bash
sudo mount -a
```

-----

## 6\. Maintenance & Reversal

### How to Monitor

  * **On Dell:** Check logs with `journalctl -u nfs-server`.
  * **On Dell:** See connected clients with `netstat -an | grep 2049`.

### How to Uninstall / Revert

If you want to stop using the Dell as a server:

1.  **Remove from Proxmox:** Go to Datacenter \> Storage \> Dell-Storage \> Remove.
2.  **Stop Service on Dell:**
    ```bash
    sudo systemctl stop nfs-kernel-server
    sudo systemctl disable nfs-kernel-server
    ```
3.  **Clean Config:**
      * Remove the line from `/etc/exports`.
      * Remove the rule from UFW: `sudo ufw delete allow from <HP_IP>`.

<!-- end list -->

```
```
