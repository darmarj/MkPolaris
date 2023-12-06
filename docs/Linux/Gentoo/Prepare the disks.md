# Prepare the disks
GUID Partition Table - he GUID Partition Table (GPT) setup (also called GPT disklabel) uses 64-bit identifiers for the partitions. The location in which it stores the partition information is much bigger than the 512 bytes of the MBR partition table (DOS disklabel), which means there is practically no limit on the amount of partitions for a GPT disk. Also the size of a partition is bounded by a much greater limit (almost 8 ZiB - yes, zebibytes).

## Default partitioning scheme
| Partition   | Filesystem   | Size  |  Description   |
| ----------- | -------------|-------|----------------|
| `/dev/sda1`   | fat32 (UEFI) or ext4 (BIOS - aka Legacy boot)   | 256M  | Boot/EFI system partition|
| `/dev/sda2`   | swap   |  RAM size \*2    | Swap partition  |
| `/dev/sda3`   | ext4   |  Rest of the disk    | Root partition  |

## Filesystems
[**btrfs**](https://wiki.gentoo.org/wiki/Btrfs)
!!! note ""
    A next generation filesystem that provides many advanced features such as snapshotting, self-healing through checksums, transparent compression, subvolumes, and integrated RAID. Kernels prior to 5.4.y are not guaranteed to be safe to use with btrfs in production because fixes for serious issues are only present in the more recent releases of the LTS kernel branches. Filesystem corruption issues are common on older kernel branches, with anything older than 4.4.y being especially unsafe and prone to corruption. Corruption is more likely on older kernels (than 5.4.y) when compression is enabled. RAID 5/6 and quota groups unsafe on all versions of btrfs. Furthermore, btrfs can counter-intuitively fail filesystem operations with ENOSPC when df reports free space due to internal fragmentation (free space pinned by DATA + SYSTEM chunks, but needed in METADATA chunks). Additionally, a single 4K reference to a 128M extent inside btrfs can cause free space to be present, but unavailable for allocations. This can also cause btrfs to return ENOSPC when free space is reported by **df**. Installing [sys-fs/btrfsmaintenance](https://packages.gentoo.org/packages/sys-fs/btrfsmaintenance) and configuring the scripts to run periodically can help to reduce the possibility of ENOSPC issues by rebalancing btrfs, but it will not eliminate the risk of ENOSPC when free space is present. Some workloads will never hit ENOSPC while others will. If the risk of ENOSPC in production is unacceptable, you should use something else. If using btrfs, be certain to avoid configurations known to have issues. With the exception of ENOSPC, information on the issues present in btrfs in the latest kernel branches is available at the btrfs [status page](https://btrfs.readthedocs.io/en/latest/Status.html).

| Filesystem    | Creation command    | On minimal CD？    | Package   |
| --------      | ---------------     | ---------------    |  ------   |
| btrfs     | mkfs.btrfs    | Yes   | [sys-fs/btrfs-progs](https://packages.gentoo.org/packages/sys-fs/btrfs-progs)   |
| ext4      | mkfs.ext4     | Yes   | [sys-fs/e2fsprogs](https://packages.gentoo.org/packages/sys-fs/e2fsprogs)     |

- EFI system partition partition (/dev/sda1) as FAT32 and the root partition (/dev/sda3) as ext4 as used in the example partition structure
```shell
root #mkfs.vfat -F 32 /dev/sda1
root #mkfs.ext4 /dev/sda3
root #mkswap /dev/sda2
```
- To activate the swap partition
```shell
root #swapon /dev/sda2
```

- Mounting the root partition
```shell
root #mkdir --parents /mnt/gentoo
root #mount /dev/sda3 /mnt/gentoo
```
