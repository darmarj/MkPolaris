---
date: 2023-12-18
authors: [darmarj]
description: >
  Kernel Warning: MDS CPU bug present and SMT on, data leak possible
categories:
  - Proxmox
---

# The issue on Proxmox 8:
MDS CPU bug present and SMT on, data leak possible. [More Detials](https://www.kernel.org/doc/html/latest/admin-guide/hw-vuln/mds.html) refer on Kernel.org

## Reproduced Condistion:
* Proxmox platform always reboot automatically espcially when trigger the vm no matter the CPUs or Memory in which assign on the configuration.
* Startup and work on for multiple VMs.
* Login to proxmox recovery mode.
* Keep VM running over night on UTF-8 time zone.
* Host hardware power on to grub mode for system preparation.

## Work around:
* Disable the option on BIOS for intel hyper-threading if it has.
* Add "mitigations=off" feature to grub for ext file system.
```bash
$ #/etc/default/grub
$ GRUB_CMDLINE_LINUX_DEFAULT="quiet mitigations=off"

$ update-grub
$ reboot
```
* Modify for zfs file system.
```bash
$ #/etc/kernel/cmdfile
$ root=ZFS=rpool/ROOT/pve-1 boot=zfs mitigations=off

$ proxmox-boot-tool refresh
4 reboot
```
