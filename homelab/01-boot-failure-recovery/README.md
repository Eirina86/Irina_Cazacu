# Case Study: Recovering a Home Server from a GRUB Boot Failure

**System:** Dell Inspiron 3793 running Ubuntu 26.04.1 LTS (kernel 7.0.0-42), self-hosted home server
**Date:** 29 October 2026
**Outcome:** Server restored to service the same evening, investigated remotely over SSH from a phone, and hardened on-site the next morning

---

## Summary

After a routine update-and-restart, the server stopped booting and halted at the GRUB bootloader with repeated `error: command failed` messages. Using only the GRUB command line, BIOS setup and photos of the screen, I diagnosed the disk layout, booted the system manually, repaired the bootloader, found an alternative boot path that worked, and finished the investigation remotely once the server was back online.

## Symptoms

- GRUB menu displayed, but selecting **Ubuntu** produced a wall of `error: command failed` followed by `error: you need to load the kernel first`
- **Advanced options** failed the same way
- After a GRUB reinstall, the system dropped straight to a `grub>` prompt where every command failed, including `echo` and `set`

## Investigation

### 1. Ruled out Secure Boot
Checked the Dell BIOS: Secure Boot was already disabled, so signature checks were not the cause.

### 2. Inspected the boot entry
Pressed `e` on the GRUB entry to view its commands. The kernel and initrd paths and root UUID looked correct, so the configuration was pointing at the right files.

### 3. Mapped the disks from the GRUB shell
```
insmod part_gpt
insmod ext2
ls
ls (hd1,gpt1)
```
This identified two drives and four partitions:

| GRUB name | Type | Role |
|---|---|---|
| `(hd0,gpt1)` | ext4, label `data` | Data drive |
| `(hd1,gpt1)` | ext4 | Ubuntu root |
| `(hd1,gpt2)` | FAT, 1.1 GB | EFI partition (mounted as `/boot/efi`) |
| `(hd1,gpt3)` | FAT, 1 GB | Second EFI partition |

Lesson learned: a UUID read from a blurry photo was mistranscribed (`e91c` vs `e31c`), causing a false "no such device" error. Using `probe --fs-uuid --set=u` to capture the UUID into a variable removed the risk of typos.

### 4. Verified the kernel files
```
set root=(hd1,gpt1)
ls -l /boot/
```
Both the current kernel (7.0.0-42) and the previous one (7.0.0-38) were present with normal file sizes, so the kernel was not missing or truncated.

### 5. Booted manually
```
probe --fs-uuid --set=u (hd1,gpt1)
linux /boot/vmlinuz-7.0.0-42-generic root=UUID=$u ro
initrd /boot/initrd.img-7.0.0-42-generic
boot
```
The system booted normally, which confirmed the kernel and root filesystem were healthy and the fault was in the boot chain.

### 6. Repaired the bootloader from inside Ubuntu
```
sudo dpkg --configure -a
findmnt /boot/efi
sudo apt install --reinstall grub-efi-amd64 shim-signed
sudo update-grub
```
GRUB (2.14-2ubuntu1) reinstalled with no errors and detected both kernels. I also configured unattended booting, which matters for a headless server:
```
GRUB_TIMEOUT=5
GRUB_RECORDFAIL_TIMEOUT=5
```
`GRUB_RECORDFAIL_TIMEOUT` stops Ubuntu waiting indefinitely at the menu after a crash or power cut.

The reboot still failed, now dropping to a `grub>` prompt where every command reported failure.

### 7. Found a working boot path
The BIOS boot list showed two firmware fallback entries. Booting the one on the second EFI partition (`F12` → `... 1 2`) reached Ubuntu normally.

### 8. Remote analysis over SSH
With the server back online, I continued from my phone over SSH:
```
sudo efibootmgr -v
sudo md5sum /boot/efi/EFI/ubuntu/* /mnt/efi3/EFI/ubuntu/*
```
Findings:
- The firmware had four boot entries across the two EFI partitions
- **All bootloader files on both partitions were byte-for-byte identical**, and both pointed at the correct root UUID

So the difference between a failing and a working boot was not the files on disk. That ruled out a corrupted bootloader and narrowed the cause to firmware-level behaviour.

### 9. On-site verification and preventive change (30 October)
- Rebooted on-site: the system went straight to the login screen with no errors and no keypress needed.
- Leading hypothesis: the **TPM measurement step**. Recent GRUB versions record each command into the TPM, and a failure there can make every command report an error, which matches the symptoms (even `echo` and `set` failing, identical files sometimes booting and sometimes not).
- Confirmed nothing depends on the TPM (`lsblk -f` showed no `crypto_LUKS`; root, data and swap are unencrypted).
- Disabled the TPM as a preventive measure. On the Inspiron 3793 it appears as **Intel® Platform Trust Technology** under *BIOS → Security* (visible with the *Advanced* toggle on), not as "TPM 2.0".
- Rebooted again: clean, unattended boot. Confirmed over SSH that the OS no longer sees a TPM (`ls /dev/tpm*` → *No such file or directory*).

## Outcome

- Server boots unattended and is reachable remotely
- Intel PTT (firmware TPM) disabled to remove the suspected failure point
- Boot order set to the entry that booted successfully: `sudo efibootmgr -o 0003,0001`
- Working EFI files backed up to `~/efi3-backup`
- Unattended boot timeouts configured (`GRUB_TIMEOUT=5`, `GRUB_RECORDFAIL_TIMEOUT=5`)

**Note on root cause:** the fault was intermittent and stopped before the TPM change, so disabling it is a preventive fix rather than a proven one. If the errors ever return with PTT off, the TPM can be ruled out and the next step would be checking for a GRUB update or a Dell BIOS update.

## Skills demonstrated

- Structured diagnosis: ruled out causes one layer at a time (Secure Boot → config → disk layout → kernel files → bootloader → firmware)
- GRUB command-line recovery and manual kernel booting
- UEFI boot management with `efibootmgr` and the Dell BIOS
- Using checksums to test a hypothesis rather than assume it
- Safe remote administration: chose not to reboot remotely when a failed boot couldn't be recovered without physical access
- Documenting changes and keeping a backup before modifying boot configuration
