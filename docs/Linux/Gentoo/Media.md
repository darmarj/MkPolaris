# Choosing the media

## Hardware requirements
AMD64 livedisk hardware requirements

|           |   Minimal CD  |   LiveDVD     |
|   ---     |   ----------  |   -------     |
|   CPU     |   Any x86-64 CPU, both AMD64 and Intel 64     |       |
|   Memory  |   2 GB    |        |
|   Disk space |  8 GB(excluding swap space)    |        |
|   Swap space  |   At least 2 GB   |        |

## Gentoo Linux installation Media
### Minimal installation
The Gentoo minimal installation CD is a bootable image: a self-contained Gentoo environment. It allows the user to boot Linux from the CD or other installation media. During the boot process the hardware is detected and the appropriate drivers are loaded. The image is maintained by Gentoo developers and allows anyone to install Gentoo if an active Internet connection is available.

The Minimal Installation CD is called install-amd64-minimal-<release>.iso.

### The occasional Gentoo LiveDVD
Occasionally, a special DVD image is crafted which can be used to install Gentoo. The instructions in this chapter target the Minimal Installation CD, so things might be a bit different when booting from the LiveDVD. However, the LiveDVD (or any other official Gentoo Linux environment) supports getting a root prompt by just invoking sudo su - or sudo -i in a terminal.

### What are stages then?
A stage3 tarball is an archive containing a profile specific minimal Gentoo environment. Stage3 tarballs are suitable to continue the Gentoo installation using the instructions in this handbook. Previously, the handbook described the installation using one of three [stage tarballs](https://wiki.gentoo.org/wiki/Stage_tarball). Gentoo does not offer stage1 and stage2 tarballs for download any more since these are mostly for internal use and for bootstrapping Gentoo on new architectures.

Stage3 tarballs can be downloaded from releases/amd64/autobuilds/ on any of the [official Gentoo mirrors](https://www.gentoo.org/downloads/mirrors/). Stage files update frequently and are not included in official installation images.

* A .CONTENTS file which is a text file listing all files available on the installation media. This file can be useful to verify if particular firmware or drivers are available on the installation media before downloading it.
* A .DIGESTS file which contains the hash of the ISO file itself, in various hashing formats/algorithms. This file can be used to verify if the downloaded ISO file is corrupt or not.
* A .asc file which is a cryptographic signature of the ISO file. This can be used to both verify if the downloaded ISO file is corrupt or not, as well as verify that the download is indeed provided by the Gentoo Release Engineering team and has not been tampered with.

### Using dd to write the ISO image to a USB driver
??? warning
    The dd command will wipe all data from the destination drive. Always backup all important data.

```shell
root #dd if=/path/to/image.iso of=/dev/sdc bs=8192k; sync
```

## Creating bootable LiveUSB drives from Linux systems
### Automatic drive-wide installation script
??? warning
    This script will **erase all data** from the USB drive. Make sure to backup any pre-existing data first, and always backup all important data.

!!! note
    ```shell
    #!/bin/bash
    set -e
    image=${1:?Supply the .iso image of a Gentoo installation medium}
    target=${2:?Supply the target device}

    echo Checking for the necessary tools presence...
    which syslinux
    which sfdisk
    which mkfs.vfat

    echo Mounting Gentoo CD image...
    cdmountpoint=/mnt/gentoo-cd
    mkdir -p "$cdmountpoint"
    trap 'echo Unmounting Gentoo CD image...; umount "$cdmountpoint"' EXIT
    mount -o loop,ro "$image" "$cdmountpoint"

    echo Creating a disk-wide EFI FAT partition on "$target"...
    echo ',,U,*' | sfdisk --wipe always "$target"

    echo Installing syslinux MBR on "$target"...
    dd if=/usr/share/syslinux/mbr.bin of="$target"
    sleep 1

    echo Creating file system on "$target"1...
    mkfs.vfat "$target"1 -n GENTOO

    echo Mounting file system...
    mountpoint=/mnt/gentoo-usb
    mkdir -p "$mountpoint"
    mount "$target"1 "$mountpoint"

    echo Copying files...
    cp -r "$cdmountpoint"/* "$mountpoint"/
    mv "$mountpoint"/isolinux/* "$mountpoint"
    mv "$mountpoint"/isolinux.cfg "$mountpoint"/syslinux.cfg
    rm -rf "$mountpoint"/isolinux*
    mv "$mountpoint"/memtest86 "$mountpoint"/memtest
    sed -i -e "s:cdroot:cdroot slowusb:" -e "s:kernel memtest86:kernel memtest:" "$mountpoint"/syslinux.cfg

    echo Unmounting file system...
    umount "$mountpoint"

    echo Installing syslinux on "$target"1
    syslinux "$target"1

    echo Syncing...
    sync

    echo 'Done!'
    ```
