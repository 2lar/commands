# Commands for everything to learn about / Ubuntu HomeLab

## 🧠 Basic Terminal / Shell Management

| Command | Purpose |
| :--- | :--- |
| `history -c` | Clears current **shell session history** (only affects the current terminal window). |
| `> ~/.bash_history` | Clears the **history file** (the permanent record of commands). |
| `history -c && > ~/.bash_history && history -w` | Clears both session and file, then writes the empty session to the file (complete wipe). |
| `groups` | Shows all **user groups** you belong to (e.g., to confirm you are in the `docker` group). |
| `sudo reboot` | **Reboots** the server (necessary after kernel updates or permission changes). |

---

## 🔐 Networking & File Sharing (Samba / Tailscale)

| Command | Purpose |
| :--- | :--- |
| `sudo apt install samba -y` | Installs the **Samba file sharing service**. |
| `sudo service smbd restart` | **Restarts the Samba service** (run this after editing `/etc/samba/smb.conf`). |
| `sudo smbpasswd -a username` | Sets or changes the **network-only password** for the Samba user. |
| `sudo smbpasswd -e username` | **Enables** the specified user in the Samba database. |
| `testparm` | Checks the `/etc/samba/smb.conf` file for **syntax errors**. |
| `tailscale ip -4` | Displays the server's **private 100.x.y.z IP address** on the Tailscale network. |
| `sudo tailscale up` | **Authenticates/Connects** the server to your Tailscale network. |

---

## 🐳 Docker Management

| Command | Purpose |
| :--- | :--- |
| `sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin` | Installs the complete **Docker stack** (Engine, CLI, and Compose plugin). |
| `sudo usermod -aG docker username` | **Grants non-sudo permissions** to run Docker commands (requires logging out/in). |
| `docker run hello-world` | **Tests the Docker installation** by running a simple container. |
| `docker ps -a` | Lists **all containers** (running and stopped). |
| `docker stop $(docker ps -aq)` | Stops **all currently running containers**. |
| `sudo systemctl restart docker` | Restarts the main **Docker service** (use after system updates). |

---

## ⚙️ System Maintenance & Diagnostics

| Command | Purpose |
| :--- | :--- |
| `sudo apt update && sudo apt upgrade -y` | **Updates** the list of packages and installs available upgrades. |
| `sudo apt autoremove` | **Removes unnecessary** (orphaned) packages (like `libllvm19`). |
| `htop` | Opens the **live process/resource monitor** (CPU, RAM usage). |
| `sudo apt install smartmontools` | Installs the tools to check **HDD/SSD health**. |
| `sudo smartctl -a /dev/sda` | Checks the **S.M.A.R.T. status** (health) of your main hard drive. |
| `sudo chown -R user:user /path/to/share` | **Changes ownership** of files/folders recursively (e.g., before sharing files). |
| `sudo chmod -R 770 /path/to/share` | Sets **read/write permissions** for owner/group (`770`) recursively. |
