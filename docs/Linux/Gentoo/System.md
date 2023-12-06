# Configuring the system
## Filesystem information

### Creating the fstab file
The /etc/fstab file uses a table-like syntax. Every line consists of six fields, separated by whitespace (space(s), tabs, or a mixture of the two). Each field has its own meaning:

1. The first field shows the block special device or remote filesystem to be mounted. Several kinds of device identifiers are available for block special device nodes, including paths to device files, filesystem labels and UUIDs, and partition labels and UUIDs.
2. The second field shows the mount point at which the partition should be mounted.
3. The third field shows the type of filesystem used by the partition.
4. The fourth field shows the mount options used by mount when it wants to mount the partition. As every filesystem has its own mount options, so system admins are encouraged to read the mount man page (man mount) for a full listing. Multiple mount options are comma-separated.
5. The fifth field is used by dump to determine if the partition needs to be dumped or not. This can generally be left as 0 (zero).
6. The sixth field is used by fsck to determine the order in which filesystems should be checked if the system wasn't shut down properly. The root filesystem should have 1 while the rest should have 2 (or 0 if a filesystem check is not necessary).

```shell
root #nano /etc/fstab
```

### Filesystem labels and UUIDs
Both MBR (BIOS) and GPT include support for filesystem labels and filesystem UUIDs. These attributes can be defined in /etc/fstab as alternatives for the mount command to use when attempting to find and mount block devices. Filesystem labels and UUIDs are identified by the LABEL and UUID prefix and can be viewed with the blkid command:

```shell
root #blkid
```

!!! danger
    If the filesystem inside a partition is wiped, then the filesystem label and the UUID values will be subsequently altered or removed.

Because of uniqueness, readers that are using an MBR-style partition table are recommended to use UUIDs over labels to define mountable volumes in /etc/fstab.

!!! info
    UUIDs of the filesystem on a LVM volume and its LVM snapshots are identical, therefore using UUIDs to mount LVM volumes should be avoided.

### Partition labels and UUIDs
Users who have gone the GPT route have a couple more 'robust' options available to define partitions in /etc/fstab. Partition labels and partition UUIDs can be used to identify the block device's individual partition(s), regardless of what filesystem has been chosen for the partition itself. Partition labels and UUIDs are identified by the PARTLABEL and PARTUUID prefixes respectively and can be viewed nicely in the terminal by running the blkid command:

```shell
root #blkid
```

Below is a more elaborate example of an /etc/fstab file:

!!! abstract "A full /etc/fstab example"
    <span class="message">FILE</span> /etc/fstab

     Adjust any formatting difference and additional partitions created from the Preparing the disks step

     |  filesystem  |   dir     |    type   |   options             |   jump    |   pass    |
     |  --------    |   -----   |    ----   |   ----------------    |   ----    |   ----    |
     |  /dev/sda1   |   /boot   |    vfat   |   defaults,noatime    |   0       |   2       |
     |  /dev/sda2   |   none    |    swap   |    sw                 |   0       |   0       |
     |  /dev/sda3   |   /       |    ext4   |    noatime            |   0       |   1       |
     |  /dev/cdrom  |   /mnt/cdrom  |   auto    |    noauto,user    |   0       |   0       |

When {++auto++} is used in the third field, it makes the **mount** command guess what the filesystem would be. This is recommended for removable media as they can be created with one of many filesystems. The {++user++} option in the fourth field makes it possible for non-root users to mount the CD.

To improve performance, most users would want to add the {++noatime++} mount option, which results in a faster system since access times are not registered (those are not needed generally anyway). This is also recommended for systems with solid state drives (SSDs).

??? tip
    Due to degradation in performance, defining the discard mount option in /etc/fstab is not recommended. It is generally better to schedule block discards on a periodic basis using a job scheduler such as cron or a timer (systemd). See Periodic fstrim jobs for more information.

## Networking information
For systems running OpenRC, a more detailed reference for network setup is available in the [advanced network configuration](https://wiki.gentoo.org/wiki/Handbook:AMD64/Networking/Introduction) section, which is covered near the end of the handbook. Systems with more specific network needs may need to skip ahead, then return here to continue with the rest of the installation.

For more specific systemd network setup, please review see the [networking portion](https://wiki.gentoo.org/wiki/Systemd#Network) of the [systemd](https://wiki.gentoo.org/wiki/Systemd) article.

### Hostname
#### Set the hostname (OpenRC or systemd)

```shell
root #echo tux > /etc/hostname
```

#### systemd

```shell
root #hostnamectl hostname tux
```

### Network
#### DHCP via dhcpcd (any init system)

```shell
root #emerge --ask net-misc/dhcpcd
```

To enable and then start the service on OpenRC systems:

```shell
root #rc-update add dhcpcd default
root #rc-service dhcpcd start
```

To enable and start the service on systmd system:

``` shell
root #systemctl enable --now dhcpcd
```

All networking information is gathered in /etc/conf.d/net. It uses a straightforward - yet perhaps not intuitive - syntax. Do not fear! Everything is explained below. A fully commented example that covers many different configurations is available in /usr/share/doc/netifrc-*/net.example.bz2.
First install [net-misc/netifrc](https://packages.gentoo.org/packages/net-misc/netifrc)

```shell
root #emerge --ask --noreplace net-misc/netifrc
```

If the network connection needs to be configured because of specific DHCP options or because DHCP is not used at all, then open /etc/conf.d/net:

```shell
root #nano /etc/conf.d/net
```

Set both config_eth0 and routes_eth0 to enter IP address information and routing information:

!!! note ""
    This assumes that the network interface will be called eth0. This is, however, very system dependent. It is recommended to assume that the interface is named the same as the interface name when booted from the installation media if the installation media is sufficiently recent. More information can be found in the [Network interface naming](https://wiki.gentoo.org/wiki/Handbook:AMD64/Networking/Advanced#Network_interface_naming) section.

!!! abstract "Static IP definition"
    <span class="skype">FILE</span>  /etc/conf.d/net

    config_eth0="192.168.0.2 netmask 255.255.255.0 brd 192.168.0.255"

    routes_eth0="default via 192.168.0.1"

To use DHCP, define config_eth0:

!!! abstract "DHCP definition"
    <span class="skype">FILE</span>  /etc/conf.d/net

    config_eth0="dhcp"

Go over /usr/share/doc/netifrc-*/net.example.bz2 for a list of additional configuration options. Be sure to also read up on the DHCP client man page if specific DHCP options need to be set.

#### Automatically start networking at boot

```shell
root #cd /etc/init.d
root #ln -s net.lo net.eth0
root #rc-update add net.eth0 default
```

!!! infor
    If the system has several network interfaces, then the appropriate net.* files need to be created just like we did with net.eth0.

    If, after booting the system, it is discovered the network interface name (which is currently documented as eth0) was wrong, then execute the following steps to rectify:

    1. Update the /etc/conf.d/net file with the correct interface name (like enp3s0 or enp5s0, instead of eth0).
    2. Create new symbolic link (like /etc/init.d/net.enp3s0).
    3. Remove the old symbolic link (**rm /etc/init.d/net.eth0**).
    4. Add the new one to the default runlevel.
    5. Remove the old one using **rc-update del net.eth0 default**.

### The hosts file

```shell
root #nano /etc/hosts
```

!!! abstract "Filling in the network information"
    <span class="skype">FILE</span>  /etc/hosts

     # This defines the current system and must be set

     127.0.0.1     tux.homenetwork tux localhost

     # Optional definition of extra systems on the network

     192.168.0.5   jenny.homenetwork jenny
     192.168.0.6   benny.homenetwork benny

## System information
### Root password

```shell
root #passwd
```

### Init and boot configuration
When using OpenRC with Gentoo, it uses /etc/rc.conf to configure the services, startup, and shutdown of a system. Open up /etc/rc.conf and enjoy all the comments in the file. Review the settings and change where needed.

#### OpenRC
```shell
root #nano /etc/rc.conf
```

Next, open /etc/conf.d/keymaps to handle keyboard configuration. Edit it to configure and select the right keyboard.

```shell
root #nano /etc/conf.d/keymaps
```

Take special care with the keymap variable. If the wrong keymap is selected, then weird results will come up when typing on the keyboard.

Finally, edit /etc/conf.d/hwclock to set the clock options. Edit it according to personal preference.

```shell
root #nano /etc/conf.d/hwclock
```

If the hardware clock is not using UTC, then it is necessary to set clock="local" in the file. Otherwise the system might show clock skew behavior.
systemd

#### systemd
First, it is recommended to run systemd-firstboot which will prepare various components of the system are set correctly for the first boot into the new systemd environment. The passing the following options will include a prompt for the user to set a locale, timezone, hostname, root password, and root shell values. It will also assign a random machine ID to the installation:

```shell
root #systemd-firstboot --prompt --setup-machine-id
root #systemctl preset-all --preset-mode=enable-only
root #systemctl preset-all
```
