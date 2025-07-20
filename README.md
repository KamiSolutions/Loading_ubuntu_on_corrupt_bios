# 🛠️ Recovering from a Corrupted BIOS Boot on Linux (UEFI)

## 📌 Background

My Ubuntu system became unbootable due to a corrupted GRUB bootloader caused by BIOS/UEFI boot mode mismatches and boot partition issues. GRUB errors like the following appeared:

```
grub-install: error: /usr/lib/grub/x86_64-efi/modinfo.sh doesn't exist.
```

or

```
grub-install: warning: EFI variables cannot be set on this system.
```

This prevented the system from booting properly.

---

## 🧩 Problem Summary

- BIOS was set to **Legacy Boot**, but the disk used **GPT with UEFI** partitioning.
- The EFI partition wasn’t mounted correctly.
- GRUB reinstallation failed due to missing EFI files or incorrect mount paths.
- EFI partition was on `/dev/sdb2`, not the main disk.

---

## ✅ Full Recovery Steps

Performed using a live Ubuntu USB environment.

### 1. Identify Partitions

```bash
sudo fdisk -l
```

- Found root partition: `/dev/sda2`
- Found EFI partition: `/dev/sdb2`

### 2. Mount System for `chroot`

```bash
sudo mount /dev/sda2 /mnt
sudo mkdir -p /mnt/boot/efi
sudo mount /dev/sdb2 /mnt/boot/efi

sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys

sudo chroot /mnt
```

### 3. Install UEFI GRUB Packages

```bash
apt update
apt install grub-efi-amd64 grub-efi-amd64-signed efibootmgr shim-signed mokutil
```

> Note: This removed legacy GRUB (grub-pc) and installed UEFI-compatible packages.

### 4. Install GRUB for UEFI

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu --recheck
```

> Warning may appear: `EFI variables cannot be set on this system` — ignore this if installation completes successfully.

### 5. Update GRUB Config

```bash
update-grub
```

---

## 🔄 Final Steps

- Rebooted into BIOS setup
- Set boot mode to **UEFI only**
- Disabled **Legacy Boot** and **CSM** (Compatibility Support Module)
- Booted into Ubuntu successfully 🎉

---

## 🧠 Lessons Learned

- Always match your BIOS boot mode to your partition style (UEFI with GPT).
- The EFI partition must be mounted before installing GRUB.
- Even major boot issues can be fixed using a Live USB and `chroot`.

---

> This document is a step-by-step recovery log shared for others facing similar GRUB/EFI boot issues.

---

### 💬 Feel free to contribute or suggest improvements to this guide.

---

