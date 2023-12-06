# Installing base system

## Selecting mirrors
In order to download source code quickly it is recommended to select a fast mirror. Portage will look in the make.conf file for the GENTOO_MIRRORS variable and use the mirrors listed therein.
```shell
root #mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
```

## Gentoo ebuild repository
A second important step in selecting mirrors is to configure the Gentoo ebuild repository via the /etc/portage/repos.conf/gentoo.conf file. This file contains the sync information needed to update the package repository (the collection of ebuilds and related files containing all the information Portage needs to download and install software packages).
```shell
root #mkdir --parents /mnt/gentoo/etc/portage/repos.conf
root #cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

```toml
# /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
[DEFAULT]
main-repo = gentoo

[gentoo]
location = /var/db/repos/gentoo
sync-type = rsync
sync-uri = rsync://rsync.gentoo.org/gentoo-portage
auto-sync = yes
sync-rsync-verify-jobs = 1
sync-rsync-verify-metamanifest = yes
sync-rsync-verify-max-age = 24
sync-openpgp-key-path = /usr/share/openpgp-keys/gentoo-release.asc
sync-openpgp-key-refresh-retry-count = 40
sync-openpgp-key-refresh-retry-overall-timeout = 1200
sync-openpgp-key-refresh-retry-delay-exp-base = 2
sync-openpgp-key-refresh-retry-delay-max = 60
sync-openpgp-key-refresh-retry-delay-mult = 4
```

## DNS info
```shell
root #cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

## Mounting the necessary filesystmes
In a few moments, the Linux root will be changed towards the new location.

The filesystems that need to be made available are:

|   Filesystem  |   Description |
|   --------    |   ----------  |
|   **/proc/**  | a pseudo-filesystem. It looks like regular files, but is generated on-the-fly by the Linux kernel |
|   **/sys/**   | is a pseudo-filesystem, like /proc/ which it was once meant to replace, and is more structured than /proc/  |
|   **/dev/**   | is a regular file system which contains all device. It is partially managed by the Linux device manager (usually udev) |
|   **/run/**   | is a temporary file system used for files generated at runtime, such as PID files or locks  |

The /proc/ location will be mounted on /mnt/gentoo/proc/ whereas the others are bind-mounted. The latter means that, for instance, /mnt/gentoo/sys/ will actually be /sys/ (it is just a second entry point to the same filesystem) whereas /mnt/gentoo/proc/ is a new mount (instance so to speak) of the filesystem.

```shell
root #mount --types proc /proc /mnt/gentoo/proc
root #mount --rbind /sys /mnt/gentoo/sys
root #mount --make-rslave /mnt/gentoo/sys
root #mount --rbind /dev /mnt/gentoo/dev
root #mount --make-rslave /mnt/gentoo/dev
root #mount --bind /run /mnt/gentoo/run
root #mount --make-slave /mnt/gentoo/run
```

!!! note ""
    The <span class="rouge">--make-rslave</span> operations are needed for systemd support later in the installation.

## Entering the new environment
Now that all partitions are initialized and the base environment installed, it is time to enter the new installation environment by chrooting into it. This means that the session will change its root (most top-level location that can be accessed) from the current installation environment (installation CD or other installation medium) to the installation system (namely the initialized partitions). Hence the name, change root or chroot.

This chrooting is done in three steps:

1.  The root location is changed from / (on the installation medium) to /mnt/gentoo/ (on the partitions) using chroot
2.  Some settings (those in /etc/profile) are reloaded in memory using the source command
3.  The primary prompt is changed to help us remember that this session is inside a chroot environment.

```shell
root #chroot /mnt/gentoo /bin/bash
root #source /etc/profile
root #export PS1="(chroot) ${PS1}"
```

## Mounting the boot partition
```shell
root #mount /dev/sda1 /boot
```

## Congiuring Portage
Installing a Gentoo ebuild repository snapshot from the web
