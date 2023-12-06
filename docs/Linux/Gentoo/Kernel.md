# Configuring the kernel

## Installing firmware and/or microcode

### Firmware
Before getting to configuring kernel sections, it is beneficial to be aware that some hardware devices require additional, sometimes non-FOSS compliant, firmware to be installed on the system before they will operate correctly. This is often the case for wireless network interfaces commonly found in both desktop and laptop computers. Modern video chips from vendors like AMD, Nvidia, and Intel, often also require external firmware files to be fully functional. Most firmware for modern hardware devices can be found within the [sys-kernel/linux-firmware](https://packages.gentoo.org/packages/sys-kernel/linux-firmware) package. This is the recommendation to have to be installed before the initial system reboot.

```shell
root #emerge --ask sys-kernel/linux-firmware
```

!!! note ""
    Installing certain firmware packages often requires accepting the associated firmware licenses. If necessary, visit the license handling section of the Handbook for help on accepting licenses.

###  Microcode
In addition to discrete graphics hardware and network interfaces, CPUs also can require firmware updates. Typically this kind of firmware is referred to as microcode. Newer revisions of microcode are sometimes necessary to patch instability, security concerns, or other miscellaneous bugs in CPU hardware.

Microcode updates for AMD CPUs are distributed within the aforementioned sys-kernel/linux-firmware package. Microcode for Intel CPUs can be found within the [sys-firmware/intel-microcode](https://packages.gentoo.org/packages/sys-firmware/intel-microcode) package, which will need to be installed separately. See the {++[Microcode article](https://wiki.gentoo.org/wiki/Microcode)++} for more information on how to apply microcode updates.

## Kernel configuration and compilation
- [Full automation approach: Distribution kernels](#jump)
- [Hybrid approach: Genkernel](#2)
- [Full manual approach](#3)

### <span id="jump">Distribution kernels Distribution kernels</span>
Distribution Kernels are ebuilds that cover the complete process of unpacking, configuring, compiling, and installing the kernel. The primary advantage of this method is that the kernels are updated to new versions by the package manager as part of @world upgrade. This requires no more involvement than running an emerge command. Distribution kernels default to a configuration supporting the majority of hardware, however two mechanisms are offered for customization: savedconfig and config snippets. See the project page for more details on configuration.

#### Installing the correct installkernel package
```shell
# systemd-boot as the bootloader
root #emerge --ask sys-kernel/installkernel-systemd-boot
```

```shell
# Grub,LILO,etc traditional layout
root #emerge --ask sys-kernel/installkernel-gentoo
```

#### Install a distribution kernel
```shell
# build a kernel with Gentoo patches form source
root #emerge --ask sys-kernel/gentoo-kernel
```

#### Upgrading and cleaning up
Once the kernel is installed, the package manager will automatically update it to newer versions. The previous versions will be kept until the package manager is requested to clean up stale packages. To reclaim disk space, stale packages can be trimmed by periodically running emerge with the --depclean option:
```shell
root #emerge --depclean

# clean up old kernel versions as specifically
emerge --prune sys-kernel/gentoo-kernel sys-kernel/gentoo-kernel-bin
```

#### Post-install/upgrade tasks
Distribution kernels are capable of rebuilding kernel modules installed by other packages. linux-mod.eclass provides the dist-kernel USE flag which controls a subslot dependency on [virtual/dist-kernel](https://packages.gentoo.org/packages/virtual/dist-kernel).

Enabling this USE flag on packages like [sys-fs/zfs](https://packages.gentoo.org/packages/sys-fs/zfs) and [sys-fs/zfs-kmod](https://packages.gentoo.org/packages/sys-fs/zfs-kmod) allows them to automatically be rebuilt against a newly updated kernel and, if applicable, will re-generate the initramfs accordingly.
Manually rebuilding the initramfs

If required, manually trigger such rebuilds by, after a kernel upgrade, executing:
```shell
root #emerge --ask @module-rebuild
```

If any kernel modules (e.g. ZFS) are needed at early boot, rebuild the initramfs afterward via:
```shell
root #emerge --config sys-kernel/gentoo-kernel
root #emerge --config sys-kernel/gentoo-kernel-bin
```

#### Installing the kernel sources
!!! note ""
    This section is only relevant when using the following **genkernel**(hybrid) or manual kernel management approach.

When installing and compiling the kernel for amd64-based systems, Gentoo recommends the [sys-kernel/gentoo-sources](https://packages.gentoo.org/packages/sys-kernel/gentoo-sources) package.

Choose an appropriate kernel source and install it using emerge:
```shell
root #emerge --ask sys-kernel/gentoo-sources
```

This will install the Linux kernel sources in /usr/src/ using the specific kernel version in the path. It will not create a symbolic link by itself without USE=symlink being enabled on the chosen kernel sources package.

It is conventional for a /usr/src/linux symlink to be maintained, such that it refers to whichever sources correspond with the currently running kernel. However, this symbolic link will not be created by default. An easy way to create the symbolic link is to utilize eselect's kernel module.

For further information regarding the purpose of the symlink, and how to manage it, please refer to Kernel/Upgrade.

First, list all installed kernels:
```shell
root #eselect kernel list

Available kernel symlink targets:
  [1]   linux-5.15.52-gentoo
```

In order to create a symbolic link called linux, use:
```shell
eselect kernel set 1
```

```shell
root #ls -l /usr/src/linux

lrwxrwxrwx    1 root   root    12 Oct 13 11:04 /usr/src/linux -> linux-5.15.52-gentoo
```


### <span id="2" class="jade">Genkernel</span>
If an entirely manual configuration looks too daunting, system administrators should consider using genkernel as a hybrid approach to kernel maintenance.

{++Genkernel provides a generic kernel configuration file, automatically generates the kernel, initramfs, and associated modules, and then installs the resulting binaries to the appropriate locations.++} {++This results in minimal and generic hardware support for the system's first boot, and allows for additional update control and customization of the kernel's configuration in the future.++}

??? info
    While using genkernel to maintain the kernel provides system administrators with more update control over the system's kernel, initramfs, and other options, it will require a time and effort commitment to perform future kernel updates as new sources are released. Those looking for a hands-off approach to kernel maintenance should use a distribution kernel.

    Misconception to believe genkernel automatically generates a custom kernel configuration for the hardware on which it is run;

    It uses a predetermined kernel configuration that supports most generic hardware and automatically handles the make commands necessary to assemble and install the kernel, the associate modules, and the initramfs file.

As a prerequisite, due to the firwmare USE flag being enabled by default for the sys-kernel/genkernel package, the package manager will also attempt to pull in the sys-kernel/linux-firmware package. The binary redistributable software licenses are required to be accepted before the linux-firmware will install.

This license group can be accepted system-wide for any package by adding the \@BINARY-REDISTRIBUTABLE as an ACCEPT_LICENSE value in the /etc/portage/make.conf file. It can be exclusively accepted for the linux-firmware package by adding a specific inclusion via a /etc/portage/package.license/linux-firmware file.

If necessary, review the [methods of accepting software licenses](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Optional:_Configure_the_ACCEPT_LICENSE_variable) available in the Installing the base system chapter of the handbook, then make some changes for acceptable software licenses.

If in analysis paralysis, the following will do the trick:
```shell
root #mkdir /etc/portage/package.license
```

!!! pied-piper "/etc/portage/package.license/linux-firmware"
    Accept binary redistributable licenses for the linux-firmware package
    ```shell
    sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE
    ```
#### Installation
Explanations and prerequisites aside, install the [sys-kernel/genkernel](https://packages.gentoo.org/packages/sys-kernel/genkernel) package:

#### Generation
Compile the kernel sources by running genkernel all. Be aware though, as genkernel compiles a kernel that supports a wide array of hardware for differing computer architectures, this compilation may take quite a while to finish.

!!! note ""
    If the root partition/volume uses a filesystem other than ext4, it may be necessary to manually configure the kernel using genkernel --menuconfig all to add built-in kernel support for the particular filesystem(s) (i.e. not building the filesystem as a module).

    Users of LVM2 should add --lvm as an argument to the genkernel command below.

```shell
root #genkernel --mountboot --install all
```

Once genkernel completes, a kernel and an initial ram filesystem (initramfs) will be generated and installed into the /boot directory. Associated modules will be installed into the /lib/modules directory. The initramfs will be started immediately after loading the kernel to perform hardware auto-detection (just like in the live disk image environments).

```shell
root #ls /boot/vmlinu* /boot/initramfs*
root #ls /lib/modules
```

### <span id="3">Manual configuration</span>
#### Introduction
Manually configuring a kernel is often seen as the most difficult procedure a Linux user ever has to perform. Nothing is less true - after configuring a couple of kernels no one remembers that it was difficult!

However, one thing is true: it is vital to know the system when a kernel is configured manually. Most information can be gathered by emerging sys-apps/pciutils which contains the lspci command:
```shell
root #emerge --ask sys-apps/pciutils
```

!!! note ""
    Inside the chroot, it is safe to ignore any pcilib warnings (like pcilib: cannot open */sys/bus/pci/devices*) that **lspci** might throw out.

Another source of system information is to run lsmod to see what kernel modules the installation CD uses as it might provide a nice hint on what to enable.

Now go to the kernel source directory and execute make menuconfig. This will fire up menu-driven configuration screen.

```shell
root #cd /usr/src/linux
root #make menuconfig
```

!!! success "Gentoo kernel configuration guide"
    The Linux kernel configuration has many, many sections. Let's first list some options that must be activated (otherwise Gentoo will not function, or not function properly without additional tweaks). We also have a [Gentoo kernel configuration guide](https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide) on the Gentoo wiki that might help out further.

#### Enabling required options
When using sys-kernel/gentoo-sources, it is strongly recommend the Gentoo-specific configuration options be enabled. These ensure that a minimum of kernel features required for proper functioning is available:

!!! pied-piper "Enable Gentoo-specific options"
    <span class="turquoise">KERNEL</span>

        Gentoo Linux --->
          Generic Driver Options --->
            [*] Gentoo Linux support
            [*]   Linux dynamic and persistent device naming (userspace devfs) support
            [*]   Select options required by Portage features
                Support for init systems, system and service manager --->
                  [*] OpenRC, runit and other script based systems and managers.
                  [*] systemd

Naturally the choice in the last two lines depends on the selected init system ([OpenRC](https://wiki.gentoo.org/wiki/OpenRC) vs. [systemd](https://wiki.gentoo.org/wiki/Systemd)). It does not hurt to have support for both init systems enabled.

#### Enabling support for typical system components
Make sure that every driver that is vital to the booting of the system (such as SATA controllers, NVMe block device support, filesystem support, etc.) is compiled in the kernel and not as a module, otherwise the system may not be able to boot completely.

Next select the exact processor type. It is also recommended to enable MCE features (if available) so that users are able to be notified of any hardware problems. On some architectures (such as x86_64), these errors are not printed to **dmesg**, but to ^^/dev/mcelog^^. This requires the [app-admin/mcelog](https://packages.gentoo.org/packages/app-admin/mcelog) package.

Also select Maintain a devtmpfs file system to mount at /dev so that critical device files are already available early in the boot process _(CONFIG_DEVTMPFS_ and _CONFIG_DEVTMPFS_MOUNT)_:

!!! pied-piper "Enable devtmpfs support(_CONFIG_DEVTMPFS)"
    <span class="turquoise">KERNEL</span>

        Device Driver --->
          Generic Driver Options --->
            [*] Maintain a devtmpfs filesystem to mount at /dev
            [*]   Automount devtmpfs at /dev, after the kernel mounted the rootfs

Verify SCSI disk support has been activated _(CONFIG_BLK_DEV_SD)_:

!!! pied-piper "Enabling SCSI disk support _(CONFIG_SCSI, CONFIG_BLK_DEV_SD)_"
    <span class="turquoise">KERNEL</span>

        Device Driver --->
          SCSI device support --->
            <*> SCSI device support
            <*> SCSI disk support

!!! pied-piper "Enabling basic SATA and PATA support _(CONFIG_ATA_ACPI, CONFIG_SATA_PMP, CONFIG_SATA_AHCI, CONFIG_ATA_BMDMA, CONFIG_ATA_SFF, CONFIG_ATA_PIIX)_"
    <span class="turquoise">KERNEL</span>

        Device Driver --->
          <*> Serial ATA and Parallel ATA drivers (libata) --->
            [*] ATA ACPI Support
            [*] SATA Poart Multiplier support
            <*> AHCI SATA support (achi)
            [*] ATA BMDMA support
            [*] ATA SFF support (for legacy IDE and PATA)
            <*> Intel ESB, ICH, PIIX3, PIIX4 PATA/SATA support (ata_piix)

Verify basic NVMe support has been enabled:
!!! pied-piper "Enable basic NVMe support for Linux 4.4.x _(CONFIG_BLK_DEV_NVME)_"
    <span class="turquoise">KERNEL</span>

        Device Driver --->
          <*> NVM Express block device

!!! pied-piper "Enable basic NVMe support for Linux 5.x.x _(CONFIG_DEVTMPFS)_"
    <span class="turquoise">KERNEL</span>

        Device Driver --->
          NVME support --->
            <*> NVM Express block device

It does not hurt to enable the following additional NVMe support:

!!! pied-piper "Enabling additional NVMe support _(CONFIG_NVME_MULTIPATH, CONFIG_NVME_MULTIPATH, CONFIG_NVME_HWMON, CONFIG_NVME_FC, CONFIG_NVME_TCP, CONFIG_NVME_TARGET, CONFIG_NVME_TARGET_PASSTHRU, CONFIG_NVME_TARGET_LOOP, CONFIG_NVME_TARGET_FC, CONFIG_NVME_TARGET_FCLOOP, CONFIG_NVME_TARGET_TCP_"
    <span class="turquoise">KERNEL</span>

        [*] NVMe multipath support
        [*] NVME hardware monitoring
        <M> NVM Express over Fabrics FC host driver
        <M> NVM Express over Fabrics TCP host driver
        <M> NVM Traget support
          [*]   NVMe Target Passthrough support
          <M>   NVMe loopback device support
          <M>   NVMe over Fabrics FC target driver
          < >     NVMe over Fabrics FC Transport Loopback Test driver (NEW)
          <M>   NVMe over Fabrics TCP target driver

Now go to File Systems and select support for the filesystems that will be used by the system. Do not compile the file system that is used for the root filesystem as module, otherwise the system may not be able to mount the partition. Also select Virtual memory and /proc file system. Select one or more of the following options as needed by the system:

!!! pied-piper "Enable file system support _(CONFIG_EXT2_FS, CONFIG_EXT3_FS, CONFIG_EXT4_FS, CONFIG_BTRFS_FS, CONFIG_MSDOS_FS, CONFIG_VFAT_FS, CONFIG_PROC_FS, and CONFIG_TMPFS)_"
    <span class="turquoise">KERNEL</span>

        File systems --->
          <*> Second extended fs support
          <*> The Extended 3 (ext3) filesystem
          <*> The Extended 4 (ext3) filesystem
          <*> Btrfs filesystem support
          DOS/FAT/NT Filesystems --->
            <*> MSDOS fs support
            <*> VFAT (Windows-95) fs support
          Pseudo Filesystems --->
            [*] /porc file system support
            [*] Tmpfs virtual memory file system support (former shm fs)

Most systems also have multiple cores at their disposal, so it is important to activate Symmetric multi-processing support (CONFIG_SMP):
!!! pied-piper "Activating SMP support _(CONFIG_SMP)_"
    <span class="turquoise">KERNEL</span>

        Processor type and features --->
          [*] Symmetric multi-processing support

If USB input devices (like keyboard or mouse) or other USB devices will be used, do not forget to enable those as well:
!!! pied-piper "Enable USB and human input device support _(CONFIG_HID_GENERIC, CONFIG_USB_HID, CONFIG_USB_SUPPORT, CONFIG_USB_XHCI_HCD, CONFIG_USB_EHCI_HCD, CONFIG_USB_OHCI_HCD, (CONFIG_HID_GENERIC, CONFIG_USB_HID, CONFIG_USB_SUPPORT, CONFIG_USB_XHCI_HCD, CONFIG_USB_EHCI_HCD, CONFIG_USB_OHCI_HCD, CONFIG_USB4)_"
    <span class="turquoise">KERNEL</span>

        Device Drivers --->
          HID support  --->
            -*- HID bus support
            <*>   Generic HID driver
            [*]   Battery level reporting for HID devices
              USB HID suport --->
                <*> USB HID transport layer
        [*] USB support --->
          <*>       xHCI HCD (USB 3.0) support
          <*>       EHCI HCD (USB 2.0) support
          <*>       OHCI HCD (USB 1.1) support
        <*> Unified support for USB4 and Thunderbolt --->


If PPPoE is used to connect to the Internet, or a dial-up modem, then enable the following options _(CONFIG_PPP, CONFIG_PPP_ASYNC, and CONFIG_PPP_SYNC_TTY)_:

!!! pied-piper "Enabling PPPoE support _(PPPoE, CONFIG_PPPOE, CONFIG_PPP_ASYNC, CONFIG_PPP_SYNC_TTY_"
    <span class="turquoise">KERNEL</span>

        Device Driver --->
          Network device support --->
            <*> PPP (point-to-point protocol) support
            <*> PPP over Ethernet
            <*> PPP support for async serial ports
            <*> PPP support for sync tty point

#### Architecture specific kernel configuration
Enable GPT partition label support if that was used previously when partitioning the disk (CONFIG_PARTITION_ADVANCED and CONFIG_EFI_PARTITION):

!!! pied-piper "Enable support for GPT"
    <span class="turquoise">KERNEL</span>

        _*_ Enable the block layer --->
           Partition Types --->
            [*] Advanced partition selection
            [*] EFI GUID Partition support

Enable EFI stub support, EFI variables and EFI Framebuffer in the Linux kernel if UEFI is used to boot the system _(CONFIG_EFI, CONFIG_EFI_STUB, CONFIG_EFI_MIXED, CONFIG_EFI_VARS, and CONFIG_FB_EFI)_:

!!! pied-piper "Enable support for UEFI"
    <span class="turquoise">KERNEL</span>

        Processor type and features --->
           [*] EFI runtime service support
           [*]   EFI stub support
           [*]     EFI mixed-mode support

        Device Drivers
            Firmware Drivers   --->
                EFI (Extension Firmware Interface) Support  --->
                    <*> EFI Veriable Support via sysfs
            Graphics support   --->
                Frame buffer Devices  --->
                    <*> Support for frame buffer devices  --->
                        [*]     EFI-based Framebuffer Support

#### Compiling and installing
With the configuration now done, it is time to compile and install the kernel. Exit the configuration and start the compilation process:
```shell
root #make && make modules_install
```

??? note
    It is possible to enable parallel builds using make -jX with X being an integer number of parallel tasks that the build process is allowed to launch. This is similar to the instructions about /etc/portage/make.conf earlier, with the MAKEOPTS variable.

When the kernel has finished compiling, copy the kernel image to /boot/. This is handled by the make install command:
```shell
root #make install
```

#### Optional: Building an initramfs
In certain cases it is necessary to build an initramfs - an initial ram-based file system. The most common reason is when important file system locations (like /usr/ or /var/) are on separate partitions. With an initramfs, these partitions can be mounted using the tools available inside the initramfs.

Without an initramfs, there is a risk that the system will not boot properly as the tools that are responsible for mounting the file systems require information that resides on unmounted file systems. An initramfs will pull in the necessary files into an archive which is used right after the kernel boots, but before the control is handed over to the init tool. Scripts on the initramfs will then make sure that the partitions are properly mounted before the system continues booting.

!!! danger "Important"
    If using genkernel, it should be used for both building the kernel and the initramfs. When using genkernel only for generating an initramfs, it is crucial to pass {++--kernel-config=/path/to/kernel.config++} to genkernel or the generated initramfs may not work with a manually built kernel. Note that manually built kernels go beyond the scope of support for the handbook. See the [kernel configuration](https://wiki.gentoo.org/wiki/Kernel/Configuration) article for more information.

To install an initramfs, install [sys-kernel/dracut](https://packages.gentoo.org/packages/sys-kernel/dracut) first, then have it generate an initramfs:

```shell
root #emerge --ask sys-kernel/dracut
root #dracut --kver=5.15.52-gentoo
```

The initramfs will be stored in /boot/. The resulting file can be found by simply listing the files starting with initramfs:

```shell
root #ls /boot/initramfs*
```

## Kernel modules
### Listing available kernel modules

!!! note ""
    Hardware modules are optional to be listed manually. **udev** will normally load all hardware modules that are detected to be connected in most cases. However, it is not harmful for modules that will be automatically loaded to be listed. Modules cannot be loaded twice; they are either loaded or unloaded. Sometimes exotic hardware requires help to load their drivers.

The modules that need to be loaded during each boot in can be added to /etc/modules-load.d/*.conf files in the format of one module per line. When extra options are needed for the modules, they should be set in /etc/modprobe.d/*.conf files instead.

To view all modules available for a specific kernel version, issue the following **find** command. Do not forget to substitute "<kernel version>" with the appropriate version of the kernel to search:

```shell
root #find /lib/modules/<kernel version>/ -type f -iname '*.o' -or -iname '*.ko' | less
```

### Force loading particular kernel modules
To force load the kernel to load the 3c59x.ko module (which is the driver for a specific 3Com network card family), edit the /etc/modules-load.d/network.conf file and enter the module name within it.

```shell
root #mkdir -p /etc/modules-load.d
root #nano -w /etc/modules-load.d/network.conf
```

Note that the module's .ko file suffix is insignificant to the loading mechanism and left out of the configuration file:

!!! abstract "Force loading 3c59x module"
    <span class="message">FILE</span> /etc/modules-load.d/network.conf

    3c59x
