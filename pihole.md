# Network Configuration: Xfinity & Pi-hole Setup

**Date:** December 12, 2025
**Environment:** Xfinity xFi Gateway (Restrictive), Ubuntu Server (Dell), Windows Client.

## 1\. The Challenge

Xfinity xFi Gateways (`10.0.0.1`) restrict user access to critical DNS settings.

  * **Restriction:** Users cannot change the DNS server on the router level.
  * **Consequence:** Network-wide ad blocking (Pi-hole) cannot be enforced automatically via the router.
  * **Solution:** Manual DNS configuration on individual clients (Side-stepping the router).

## 2\. Architecture Overview

  * **Router:** Xfinity Gateway acting as Modem/Router (Standard Mode, not Bridge Mode).
  * **DNS Server:** Dell Desktop running Ubuntu (hosting Pi-hole, NFS, Samba).
  * **Client:** Windows Desktop.

## 3\. Server Setup (Ubuntu/Dell)

### A. Static IP Reservation

To ensure the Pi-hole's address never changes, a "Reserved IP" was set on the Xfinity Gateway.

1.  Log in to `http://10.0.0.1`.
2.  Navigate to **Connected Devices**.
3.  Locate the Ubuntu machine.
4.  **Edit** -\> Change from DHCP to **Reserved IP**.

### B. Firewall Configuration

The Ubuntu firewall (UFW) must allow DNS traffic, or clients will get "Request Timed Out."

```bash
sudo ufw allow 53/tcp  # DNS
sudo ufw allow 53/udp  # DNS
sudo ufw allow 80/tcp  # Pi-hole Web Interface
```

### C. Installation Method

  * **Current State:** "Bare Metal" installation (Directly on OS).
  * **Note:** While Portainer is running on this server, Pi-hole was installed natively to avoid Docker Port 53 conflicts with Ubuntu's `systemd-resolved`.

## 4\. Client Configuration (Windows)

To force the Windows PC to use Pi-hole instead of Xfinity's DNS:

### A. Manual IPv4 DNS

1.  **Settings \> Network & Internet \> Wi-Fi/Ethernet \> Edit**.
2.  **IPv4:** On.
3.  **Preferred DNS:** `[Insert Ubuntu IP, e.g., 10.0.0.5]`.
4.  **Alternate DNS:** Left blank (to prevent fallback to ads).

### B. Disabling IPv6 (Crucial)

Windows prefers IPv6, and Xfinity pushes their own DNS via IPv6, causing "DNS Leaks" (ads still appearing).

1.  Run `ncpa.cpl` (Network Connections).
2.  Right-click Adapter -\> **Properties**.
3.  **Uncheck** "Internet Protocol Version 6 (TCP/IPv6)".
4.  **Restart** the computer.

## 5\. Verification Commands

Use these commands in the Windows Command Prompt to verify success.

**1. The "Force Test" (Is Pi-hole alive?)**
Target the specific IP to ensure it responds.

```cmd
nslookup google.com [Your_Ubuntu_IP]
```

  * *Success:* Returns IP addresses.
  * *Fail:* "Timed Out" (Check Ubuntu Firewall).

**2. The "Default Test" (Is Windows obeying?)**
Check if Windows uses the Pi-hole automatically.

```cmd
nslookup google.com
```

  * *Success:* `Server: UnKnown` (or hostname) and `Address: [Your_Ubuntu_IP]`.
  * *Fail:* `Server: cdns01.comcast.net` (IPv6 is likely still on).

**3. The "Ad Block Test"**
Check if a known ad domain returns 0.0.0.0.

```cmd
nslookup flurry.com
```

  * *Success:* Address is `0.0.0.0` or `::`.

## 6\. Safety Notes

  * **10.0.0.1 Security:** The default password (`password`) for the Xfinity Gateway was changed to a secure unique password.
  * **Bridge Mode:** Bridge mode was **avoided** to maintain Xfinity connectivity without purchasing a secondary router.
  * **Security:** This setup relies on the Xfinity firewall for edge protection, while Pi-hole handles internal DNS filtering.

## 7\. Client Configuration (Apple Devices)

Just like Windows, Apple devices must be manually pointed to the Pi-hole IP (`10.0.0.X`) to bypass the Xfinity router defaults.

### A. iOS (iPhone / iPad)

*Note: This setting is per-network. You only need to do this once for your home Wi-Fi.*

1.  Open **Settings \> Wi-Fi**.
2.  Tap the **blue "i" icon** next to your connected Xfinity network.
3.  Scroll down to **DNS \> Configure DNS**.
4.  Switch from **Automatic** to **Manual**.
5.  **Delete** any existing servers (tap the red minus circle).
6.  Tap **Add Server** and enter your Ubuntu IP (e.g., `10.0.0.5`).
7.  Tap **Save** in the top right corner.

### B. macOS (Macbook / iMac)

*Note: Steps may vary slightly between macOS Ventura/Sonoma and older versions, but the logic is the same.*

**1. Set Manual DNS**

1.  Go to **System Settings** (or System Preferences) \> **Network**.
2.  Click on your active connection (**Wi-Fi** or **Ethernet**) -\> Click **Details...** (or Advanced).
3.  Select **DNS** from the sidebar/tab.
4.  Click the **+** button under "DNS Servers".
5.  Enter your Ubuntu IP (e.g., `10.0.0.5`).
6.  **Important:** If there are other IP addresses listed (greyed out or black), make sure your Ubuntu IP is at the very top of the list.

**2. Disable IPv6 (To prevent leaks)**
Just like Windows, macOS might try to bypass your Pi-hole using Xfinity's IPv6.

1.  Still in the **Network \> Details/Advanced** window, select **TCP/IP** from the sidebar/tab.
2.  Find the "Configure IPv6" dropdown.
3.  Change it from "Automatically" to **Link-local only**.
      * *Why?* This prevents the Mac from getting a global IPv6 address from Xfinity, forcing all traffic through your IPv4 Pi-hole connection.
4.  Click **OK** -\> **Apply**.

### C. Verification on Mac/iOS

To confirm it worked:

1.  **iOS:** Open Safari and visit `ads-blocker.com/testing` or just browse a site usually heavy with ads.
2.  **macOS:** Open Terminal and run:
    ```bash
    scutil --dns
    ```
    *Look for "nameserver[0]" matching your Ubuntu IP.*

-----

## 8\. Pi-hole Maintenance & Troubleshooting Cheat Sheet

### Essential CLI Commands

Common commands run from the Ubuntu/Dell terminal.

| Command | Description | Use Case |
| :--- | :--- | :--- |
| `pihole -t` | Live view of the log file. | "Is Pi-hole actually receiving traffic right now?" |
| `pihole -r` | Runs the repair/reconfigure wizard. | Use if web interface is broken or permissions are messed up. |
| `pihole -f` | Flushes the daily log file. | Clear today's stats from the dashboard. |
| `pihole restartdns` | Restarts the DNS resolver. | Use if clients can't connect or after changing config files manually. |
| `pihole -a -p` | Reset the Web Interface password. | If you forgot the login password. |

### Managing Logs & Database

**Location of Files:**

  * **Daily Log (Text):** `/var/log/pihole/pihole.log` (Used for `pihole -t`)
  * **Long-term History (DB):** `/etc/pihole/pihole-FTL.db` (Used for Dashboard Graphs/Query Log)

**The "Nuclear" Clear (Delete ALL History)**
If you want to completely wipe all stats and history from the beginning of time:

```bash
# 1. Stop the service
sudo service pihole-FTL stop

# 2. Delete the database
sudo rm /etc/pihole/pihole-FTL.db

# 3. Start the service (Pi-hole creates a new empty DB automatically)
sudo service pihole-FTL start
```

### Troubleshooting: "Empty Query Log"

If the Dashboard shows "Total Queries" increasing, but the **Query Log** list is empty, it is usually a permission issue between the Web Server (`www-data`) and the Pi-hole Database.

**1. Check File Permissions**

```bash
ls -l /etc/pihole/pihole-FTL.db
# Should look like: -rw-rw---- 1 pihole pihole ...
```

**2. The Permission Fix**
Grant the web user access to the Pi-hole group.

```bash
# Add web user to pihole group
sudo usermod -a -G pihole www-data

# Restart web server
sudo service lighttpd restart

# (Optional) Manually fix file permission if still broken
sudo chmod 660 /etc/pihole/pihole-FTL.db
```

### Client-Side Testing (Windows/Mac)

Use these to verify if a specific computer is using Pi-hole.

  * **Check DNS Server:** `nslookup google.com` (Should show your Ubuntu IP).
  * **Check Ad Blocking:** `nslookup flurry.com` (Should return `0.0.0.0`).
  * **Force Check specific server:** `nslookup google.com 10.0.0.X` (Bypasses Windows settings to test Pi-hole directly).
