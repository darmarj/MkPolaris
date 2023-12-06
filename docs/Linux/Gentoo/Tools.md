# Installing tools

## System logger
### OpenRC
Some tools are missing from the stage3 archive because several packages provide the same functionality. It is now up to the user to choose which ones to install.
Gentoo offers several system logger utilities. A few of these include:

* [app-admin/sysklogd](https://packages.gentoo.org/packages/app-admin/sysklogd) - Offers the traditional set of system logging daemons. The default logging configuration works well out of the box which makes this package a good option for beginners.
* [app-admin/syslog-ng](https://packages.gentoo.org/packages/app-admin/syslog-ng) - An advanced system logger. Requires additional configuration for anything beyond logging to one big file. More advanced users may choose this package based on its logging potential; be aware additional configuration is a necessity for any kind of smart logging.
* [app-admin/metalog](https://packages.gentoo.org/packages/app-admin/metalog) - A highly-configurable system logger.

There may be other system logging utilities available through the Gentoo ebuild repository as well, since the number of available packages increases on a daily basis.
!!! tips ""
    If syslog-ng is going to be used, it is recommended to install and configure logrotate. syslog-ng does not provide any rotation mechanism for the log files. Newer versions (>= 2.0) of sysklogd however handle their own log rotation.

To install the system logger of choice, emerge it. On OpenRC, add it to the default runlevel using rc-update. The following example installs and activates [app-admin/sysklogd](https://packages.gentoo.org/packages/app-admin/sysklogd) as the system's syslog utility:

```shell
root #emerge --ask app-admin/sysklogd
root #rc-update add sysklogd default
```

### systemd
While a selection of logging mechanisms are presented for OpenRC-based systems, systemd includes a built-in logger called the systemd-journald service. The systemd-journald service is capable of handling most of the logging functionality outlined in the previous system logger section. That is to say, the majority of installations that will run systemd as the system and service manager can safely skip adding a additional syslog utilities.

See man journalctl for more details on using journalctl to query and review the systems logs.

For a number of reasons, such as the case of forwarding logs to a central host, it may be important to include redundant system logging mechanisms on a systemd-based system. This is a irregular occurrence for the handbook's typical audience and considered an advanced use case. It is therefore not covered by the handbook.

## File indexing
In order to index the file system to provide faster file location capabilities, install [sys-apps/mlocate](https://packages.gentoo.org/packages/sys-apps/mlocate).

## Remote shell access
!!! tips ""
    opensshd's default configuration does not allow root to login as a remote user. Please [create a non-root user](https://wiki.gentoo.org/wiki/FAQ#How_do_I_add_a_normal_user.3F) and configure it appropriately to allow access post-installation if required, or adjust /etc/ssh/sshd_config to allow root.

To be able to access the system remotely after installation, sshd must be configured to start on boot.
### OpenRC
To add the sshd init script to the default runlevel on OpenRC:
```shell
root #rc-update add sshd default
```

If serial console access is needed (which is possible in case of remote servers), agetty must be configured.
Uncomment the serial console section in /etc/inittab:
```shell
root #nano -w /etc/inittab

# SERIAL CONSOLES
s0:12345:respawn:/sbin/agetty 9600 ttyS0 vt100
s1:12345:respawn:/sbin/agetty 9600 ttyS1 vt100
```

### systemd
To enable the SSH server, run:
```shell
root #systemctl enable sshd
root #systemctl enable getty@tty1.service
```

## Time synchronization
It is important to use some method of synchronizing the system clock. This is usually done via the [NTP](https://wiki.gentoo.org/wiki/NTP) protocol and software. Other implementations using the NTP protocol exist, like [Chrony](https://wiki.gentoo.org/wiki/Chrony).
```shell
root #emerge --ask net-misc/chrony
```

### OpenRC
```shell
root #rc-update add chronyd default
```

### systemd
```shell
root #systemctl enable chronyd.service
root #systemctl enable systemd-timesyncd.service
```

## Filesystem tools
Depending on the filesystems used, it is necessary to install the required file system utilities (for checking the filesystem integrity, creating additional file systems etc.). Note that tools for managing ext4 filesystems ([sys-fs/e2fsprogs](https://packages.gentoo.org/packages/sys-fs/e2fsprogs)) are already installed as a part of the [@system set](https://wiki.gentoo.org/wiki/System_set_(Portage)).

The following table lists the tools to install if a certain filesystem is used:

|   Filesystem  |   Package     |
|   ----------  |   -------     |
|   Ext4        |   [sys-fs/e2fsprogs](https://packages.gentoo.org/packages/sys-fs/e2fsprogs)    |
|   XFS         |   [sys-fs/xfsprogs](https://packages.gentoo.org/packages/sys-fs/xfsprogs)      |
|   ReiserFS    |   [sys-fs/reisefsprogs](https://packages.gentoo.org/packages/sys-fs/reiserfsprogs)    |
|   JFS         |   [sys-fs/jfsutils](https://packages.gentoo.org/packages/sys-fs/jfsutils)     |
|   VFAT(FAT32,...) |   [sys-fs/dosfstools](https://packages.gentoo.org/packages/sys-fs/dosfstools)     |
|   Btrfs       |   [sys-fs/btrfs-progs](https://packages.gentoo.org/packages/sys-fs/btrfs-progs)   |
|   ZFS         |   [sys-fs/zfs](https://packages.gentoo.org/packages/sys-fs/zfs)       |

!!! info ""
    For more information on filesystems in Gentoo see the [filesystem article](https://wiki.gentoo.org/wiki/Filesystem).

## Networking tools
If networking was previously configured in the [Configuring the system](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/System) step and network setup is complete, then this 'networking tools' section can be safely skipped. In this case, proceed with the section on [Configuring a bootloader](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader).

### Installing a DHCP client

!!! warning
    Most users will need a DHCP client to connect to their network. If none was installed, then the system might not be able to get on the network thus making it impossible to download a DHCP client afterwards.
```shell
root #emerge --ask net-misc/dhcpcd
```

### Installing a PPPoE client
```shell
root #emerge --ask net-dialup/ppp
```

### Installing wireless networking tools
If the system will be connecting to wireless networks, install the [net-wireless/iw](https://packages.gentoo.org/packages/net-wireless/iw) package for Open or WEP networks and/or the [net-wireless/wpa_supplicant](https://packages.gentoo.org/packages/net-wireless/wpa_supplicant) package for WPA or WPA2 networks. iw is also a useful basic diagnostic tool for scanning wireless networks.
```shell
root #emerge --ask net-wireless/iw net-wireless/wpa_supplicant
```
