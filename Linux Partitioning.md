# 🧱 Linux Partitioning for BIOS/UEFI Recovery (Ubuntu)

## 🔍 Situation

During recovery from a failed GRUB boot, I discovered that improper partitioning and BIOS settings were the root causes. Here's a breakdown of the working partition setup and how I resolved the boot issue using **manual mounting and chroot**.

---

## 🧩 Key Problems Identified

- BIOS was set to **Legacy Boot**, but disk used **GPT partitioning** meant for **UEFI**.
- System could not boot due to mismatched mode and partition table.
- GRUB failed to install properly or boot due to EFI partition not mounted.

---

## 💽 Partition Table Overview

### `/dev/sda` - Main Internal Drive (Ubuntu Installed Here)
| # | Start   | End     | Size  | FS Type | Flags        | Description            |
|---|---------|---------|-------|---------|--------------|------------------------|
| 1 | 1049kB  | 2097kB  | 1MB   | -       | `bios_grub`  | BIOS boot partition (unused) |
| 2 | 2097kB  | 500GB   | 500GB | `ext4`  | -            | Root file system `/`   |

---

### `/dev/sdb` - Bootable USB (Live Ubuntu)
| # | Start   | End     | Size  | FS Type   | Flags               | Description          |
|---|---------|---------|-------|-----------|----------------------|----------------------|
| 1 | 32.8kB  | 6.3GB   | 6.3GB | `ISO9660` | `hidden, msftdata`   | Live boot image      |
| 2 | 6.3GB   | 6.34GB  | 5MB   | -         | `boot, esp`          | EFI System Partition |
| 4 | 6.34GB  | 62.9GB  | 56GB  | `ext4`    | -                    | Persistence storage  |

> 🔎 EFI partition was mistakenly on the USB, not internal drive

---

## 🛠️ Correct Mount Setup (Live USB + Chroot)

### 1. Mount Root System
```bash
sudo mount /dev/sda2 /mnt

2. Create & Mount EFI Directory (from USB EFI partition)

sudo mkdir -p /mnt/boot/efi
sudo mount /dev/sdb2 /mnt/boot/efi

3. Bind System Directories

sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys

4. Enter Chroot

sudo chroot /mnt

🧠 Lessons Learned

    If you're installing Ubuntu in UEFI mode, your disk must use GPT, and you must create an EFI partition (~512MB).

    The EFI partition must be on the internal drive and mounted to /boot/efi.

    A red icon in GParted often signals a read-only or locked partition.

    BIOS/UEFI boot mode must match your disk layout:

        GPT + UEFI ✅

        MBR + Legacy ✅

        GPT + Legacy ❌

🧰 Recommended Partitioning for UEFI Ubuntu Install
Mount Point	Size	Format	Description
/boot/efi	512 MB	FAT32	EFI System Partition
/	30-60 GB	ext4	Root Ubuntu OS
/home	(rest)	ext4	Optional user files
swap	2-8 GB	swap	Optional swap partition

    📌 If you find yourself stuck at boot again, verify BIOS settings, mount paths, and partition types carefully.

📬 Contributions Welcome

If you've used a different setup or have suggestions, feel free to fork and open a pull request.


If you want, I can also help you with the exact commands to save and push this file to your
