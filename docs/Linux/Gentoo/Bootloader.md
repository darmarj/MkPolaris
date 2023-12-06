# Configuring the bootloader

## Selecting a boot loader
With the Linux kernel configured, system tools installed and configuration files edited, it is time to install the last important piece of a Linux installation: the boot loader.

The boot loader is responsible for firing up the Linux kernel upon boot - without it, the system would not know how to proceed when the power button has been pressed.

For amd64, we document how to configure either [GRUB](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader#Default:_GRUB) or [LILO](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader#Alternative_1:_LILO) for BIOS based systems, and [GRUB](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader#Default:_GRUB) or [efibootmgr](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader#Alternative_2:_efibootmgr) for UEFI systems.

In this section of the Handbook a delineation has been made between emerging the boot loader's package and installing a boot loader to a system disk. Here the term emerge will be used to ask [Portage](https://wiki.gentoo.org/wiki/Portage) to make the software package available to the system. The term install will signify the boot loader copying files or physically modifying appropriate sections of the system's disk drive in order to render the boot loader activated and ready to operate on the next power cycle.

## Default: GRUB
By default, the majority of Gentoo systems now rely upon GRUB (found in the sys-boot/grub package), which is the direct successor to [GRUB Legacy](https://wiki.gentoo.org/wiki/GRUB_Legacy). With no additional configuration, GRUB gladly supports older BIOS ("pc") systems. With a small amount of configuration, necessary before build time, GRUB can support more than a half a dozen additional platforms. For more information, consult the [Prerequisites section](https://wiki.gentoo.org/wiki/GRUB#Prerequisites) of the GRUB article.

### Emerge
!!! warning
    When using an older BIOS system supporting only MBR partition tables, no additional configuration is needed in order to emerge GRUB:

```shell
root #emerge --ask --verbose sys-boot/grub
```

!!! tip
    <span class="jade">UEFI users</span>: running the above command will output the enabled GRUB_PLATFORMS values before emerging. When using UEFI capable systems, users will need to ensure GRUB_PLATFORMS="efi-64" is enabled (as it is the case by default). If that is not the case for the setup, GRUB_PLATFORMS="efi-64" will need to be added to the /etc/portage/make.conf file before emerging GRUB so that the package will be built with EFI functionality:

```shell
root #echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
root #emerge --ask sys-boot/grub
```

If GRUB was somehow emerged without enabling GRUB_PLATFORMS="efi-64", the line (as shown above) can be added to make.conf and then dependencies for the world package set can be re-calculated by passing the --update --newuse options to emerge:
```shell
root #emerge --ask --update --newuse --verbose sys-boot/grub
```

### Install
Next, install the necessary GRUB files to the /boot/grub/ directory via the grub-install command. Presuming the first disk (the one where the system boots from) is /dev/sda, one of the following commands will do:

*   BIOS:
```shell
root #grub-install /dev/sda
```

*   UEFI:
!!! warning ""
    Make sure the EFI system partition has been mounted before running grub-install. It is possible for grub-install to install the GRUB EFI file (grubx64.efi) into the wrong directory without providing any indication the wrong directory was used.
    ```shell
    root #grub-install --target=x86_64-efi --efi-directory=/boot
    ```
!!! note ""
    Modify the --efi-directory option to the root of the EFI System Partition. This is necessary if the /boot partition was not formatted as a FAT variant.
!!! warning
    If grub-install returns an error like Could not prepare Boot variable: Read-only file system, it may be necessary to remount the efivars special mount as read-write in order to succeed:
    ```shell
    root #mount -o remount,rw,nosuid,nodev,noexec --types efivarfs efivarfs /sys/firmware/efi/efivars
    ```

Some motherboard manufacturers seem to only support the /efi/boot/ directory location for the .EFI file in the EFI System Partition (ESP). The GRUB installer can perform this operation automatically with the --removable option. Verify the ESP is mounted before running the following commands. Presuming the ESP is mounted at /boot (as suggested earlier), execute:
```shell
root #grub-install --target=x86_64-efi --efi-directory=/boot --removable
```
This creates the default directory defined by the UEFI specification, and then copies the grubx64.efi file to the 'default' EFI file location defined by the same specification.

### Configure
Next, generate the GRUB configuration based on the user configuration specified in the /etc/default/grub file and /etc/grub.d scripts. In most cases, no configuration is needed by users as GRUB will automatically detect which kernel to boot (the highest one available in /boot/) and what the root file system is. It is also possible to append kernel parameters in /etc/default/grub using the GRUB_CMDLINE_LINUX variable.

To generate the final GRUB configuration, run the grub-mkconfig command:
```shell
root #grub-mkconfig -o /boot/grub/grub.cfg

Generating grub.cfg ...
Found linux image: /boot/vmlinuz-5.15.52-gentoo
Found initrd image: /boot/initramfs-genkernel-amd64-5.15.52-gentoo
done
```

The output of the command must mention that at least one Linux image is found, as those are needed to boot the system. If an initramfs is used or genkernel was used to build the kernel, the correct initrd image should be detected as well. If this is not the case, go to /boot/ and check the contents using the ls command. If the files are indeed missing, go back to the kernel configuration and installation instructions.

!!! tip
    The os-prober utility can be used in conjunction with GRUB to detect other operating systems from attached drives. Windows 7, 8.1, 10, and other distributions of Linux are detectable. Those desiring dual boot systems should emerge the [sys-boot/os-prober](https://packages.gentoo.org/packages/sys-boot/os-prober) package then re-run the grub-mkconfig command (as seen above). If detection problems are encountered be sure to read the GRUB article in its entirety before asking the Gentoo community for support.

## efibootmgr
On UEFI based systems, the UEFI firmware on the system (in other words the primary bootloader), can be directly manipulated to look for UEFI boot entries. Such systems do not need to have additional (also known as secondary) bootloaders like GRUB in order to help boot the system. With that being said, the reason EFI-based bootloaders such as GRUB exist is to extend the functionality of UEFI systems during the boot process. Using efibootmgr is really for those who desire to take a minimalist (although more rigid) approach to booting their system; using GRUB (see above) is easier for the majority of users because it offers a flexible approach when booting UEFI systems.

Remember sys-boot/efibootmgr application is not a bootloader; it is a tool to interact with the UEFI firmware and update its settings, so that the Linux kernel that was previously installed can be booted with additional options (if necessary), or to allow multiple boot entries. This interaction is done through the EFI variables (hence the need for kernel support of EFI vars).

Be sure to read through the [EFI stub](https://wiki.gentoo.org/wiki/EFI_stub) kernel article before continuing. The kernel must have specific options enabled to be directly bootable by the system's UEFI firmware. It might be necessary to recompile the kernel. It is also a good idea to take a look at the [efibootmgr](https://wiki.gentoo.org/wiki/Efibootmgr) article.
!!! note ""
    To reiterate, efibootmgr is not a requirement to boot an UEFI system. The Linux kernel itself can be booted immediately, and additional kernel command-line options can be built-in to the Linux kernel (there is a kernel configuration option called CONFIG_CMDLINE that allows the user to specify boot parameters as command-line options. Even an initramfs can be 'built-in' to the kernel.

```shell
root #emerge --ask sys-boot/efibootmgr
```

Then, create the /boot/efi/boot/ location, and then copy the kernel into this location, calling it bootx64.efi:
```shell
root #mkdir -p /boot/efi/boot
root #cp /boot/vmlinuz-* /boot/efi/boot/bootx64.efi
```

Next, tell the UEFI firmware that a boot entry called "Gentoo" is to be created, which has the freshly compiled EFI stub kernel:
```shell
root #efibootmgr --create --disk /dev/sda --part 2 --label "Gentoo" --loader "\efi\boot\bootx64.efi"
```

If an initial RAM file system (initramfs) is used, add the proper boot option to it:
```shell
root #efibootmgr -c -d /dev/sda -p 2 -L "Gentoo" -l "\efi\boot\bootx64.efi" initrd='\initramfs-genkernel-amd64-5.15.52-gentoo'
```
!!! note ""
    The use of \ as directory separator is mandatory when using UEFI definitions.

## Rebooting the system
Exit the chrooted environment and unmount all mounted partitions. Then type in that one magical command that initiates the final, true test: reboot
```shell
root #exit
cdimage ~#cd
cdimage ~#umount -l /mnt/gentoo/dev{/shm,/pts,}
cdimage ~#umount -R /mnt/gentoo
cdimage ~#reboot
```
