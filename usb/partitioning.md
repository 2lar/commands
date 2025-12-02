## 🪟 Windows Method (Using DiskPart)

The standard Windows formatting tool may not restore the full capacity because of the way the bootable drive was structured. Using the command-line tool **DiskPart** is often the most reliable method.

1.  **Open Command Prompt as Administrator:** Click the Start menu, type `cmd`, right-click on "Command Prompt," and select **Run as administrator**.
2.  **Start DiskPart:** In the command prompt window, type `diskpart` and press Enter.
3.  **List Disks:** Type `list disk` and press Enter. This will show a list of all drives. **Carefully identify your USB drive** by its size (e.g., Disk 1, Disk 2, etc.). **Selecting the wrong disk will wipe the data from that disk.**
4.  **Select Your USB Disk:** Type `select disk X` (replace `X` with the number of your USB drive) and press Enter.
5.  **Clean the Disk:** Type `clean` and press Enter. **This command completely wipes the partition table and data from the selected disk.**
6.  **Create a New Primary Partition:** Type `create partition primary` and press Enter.
7.  **Format the Partition:** Type `format fs=fat32 quick` and press Enter. (You can use `fs=ntfs quick` if you need to store files larger than 4GB, or `fs=exfat quick` for compatibility with many systems and large files).
8.  **Assign a Drive Letter (Optional):** Type `assign` and press Enter. Windows will usually assign a letter automatically.
9.  **Exit:** Type `exit` and press Enter to close DiskPart.

Your USB drive should now be restored to a single, full-capacity partition.

---

## 🐧 Linux Method (Using Disk Utility/GParted)

You can use a graphical tool like **Gnome Disks (Disk Utility)** or **GParted**.

1.  **Launch the Disk Utility** (often called "Disks" or "GParted" in the Applications menu).
2.  **Select Your USB Drive:** Choose your USB drive from the list on the left (again, identify it carefully by size). 
3.  **Delete Partitions:** Select any existing partitions on the drive and use the option to **delete** them. This will make the entire drive show as **unallocated space**.
4.  **Create a New Partition Table:** Some utilities may require you to create a new partition table on the device before making a partition (e.g., in GParted: **Device** > **Create Partition Table...** and choose **MBR** or **GPT**).
5.  **Create and Format New Partition:** Select the unallocated space and create a new partition.
    * **Size:** Use the maximum available space.
    * **File System:** Choose **FAT32** (for maximum compatibility) or **NTFS/exFAT** (for larger files or better Windows integration).
6.  **Apply Changes:** Apply the changes (in GParted, this is often a green checkmark icon).
7.  
