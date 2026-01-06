# Firmware Emulation for Alaris PCU8015 Network Communication Module (NCM)
This repository documents how to emulate the Alaris PCU8015 Network Communication Module firmware using QEMU with a generic ARM Linux kernel and an initramfs-based root filesystem.

## Files and their roles
- zImage_orig: Original Linux kernel image extracted from kernel.bin of the Alaris PCU8015 NCM firmware.
- zImage_generic: Generic ARM Linux kernel image built from the upstream Linux source https://github.com/torvalds/linux.
- rootfs.ubifs: Original root filesystem extracted from rootfs.bin in the Alaris PCU8015 NCM firmware.
- initramfs.cpio.gz: Repacked root filesystem converted from the extracted rootfs.ubifs. This initramfs format is used because it is directly supported by QEMU with -initrd.

## QEMU run command
```bash
qemu-system-arm \
  -M virt -cpu cortex-a15 -m 512M \
  -kernel zImage_generic \
  -initrd initramfs.cpio.gz \
  -nographic
  -append "console=ttyAMA0,115200 loglevel=8 earlycon=pl011,0x09000000 rdinit=/sbin/init"
```
This command boots the generic ARM Linux kernel, loads the repacked initramfs, and starts the system using /sbin/init.
All kernel logs and service output are redirected to the terminal through the PL011 serial console.

**Note**:
zImage_orig is not used for emulation.
The original kernel attempts to access fixed high physical memory addresses that are not mapped by any ARM machine model supported by QEMU, which causes early boot failure.
