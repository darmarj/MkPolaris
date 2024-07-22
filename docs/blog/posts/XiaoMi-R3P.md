---
date: 2024-07-22
authors: [darmarj]
description: >
  Install Openwrt on XiaoMi R3P
categories:
  - Router
---

# Openwrt on XiaoMi R3P

## Ready for go

*   Download Developer ROM from XiaoMi
    [ROM](http://www1.miwifi.com/miwifi_download.html)
*   Make sure to bundle router with XiaoMi App, then download SSH firmware, file the password
    [Firmware](https://d.miwifi.com/rom/ssh)
*   Download [Openwrt](https://firmware-selector.immortalwrt.org)
*   One USB with FAT32
*   One toothpick

## XiaoMi Dev Firmware

### Option 1:
Login to XiaoMi router and flush dev firmware

### Option 2:
1.  Copy XiaoMi firmware(dev) into USB.
2.  Poweroff XiaoMi router, plug in USB and press&hold "reset" button, then power router on.
3.  Waiting for light flashing to <span class="amber">__yellow__</span>, then release "reset" button.
4.  Succeed when light flashing to <span class="skype">__blue__</span>.

## XiaoMi SSH
### Refer as firmware option 2

## Login router through ssh
```bash
ssh root@192.168.31.1
```

## Flush Openwrt
```bash
scp $PATH/Openwrt.bin root@192.168.31.1:/tmp
cd /tmp
```

``` bash
# 设置环境变量
nvram set flag_try_sys1_failed=1
nvram set flag_try_sys2_failed=0
nvram set flag_boot_success=0
nvram commit # save ENV into RAM
dd if=factory.bin bs=1M count=4 | mtd write - kernel1 # invoke openwrt.bin and write into disk partition
mtd erase rootfs0 # Erase all data on rootfs0 partition
mtd erase rootfs1 # Erase all data on rootfs1 partition
mtd erase overlay # Erase all data on overlay partition
dd if=factory.bin bs=1M skip=4 | mtd write - rootfs0 # invoke openwrt.bin and write into rootfs0 partition
reboot
```
:material-google-downasaur: [dd](https://www.geeksforgeeks.org/dd-command-linux)
:material-google-downasaur: [mtd](https://openwrt.org/docs/techref/mtd)

```bash
# review system partition
cat /proc/mtd
```

## Revert to XiaoMi Official Firmware
Step 1

```bash
# ENV
fw_setenv flag_try_sys1_failed 0
fw_setenv flag_try_sys2_failed 1
fw_setenv flag_boot_success 0
```

!!! failure
    Install [Openwrt Official R3P](https://openwrt.org/toh/hwdata/xiaomi/xiaomi_mi_router_3_pro) upgrade firmware if fails do upon command for ENV. Do it again after then.

Step 2

Poweroff router, refer as [XiaoMi Dev Firmware]

  [XiaoMi Dev Firmware]: #xiaomi-dev-firmware

:material-google-downasaur: [mucen](https://mucen.cn/docs/openwrt/install/xiaomi-router-install-OpenWrt/#%E5%AE%89%E8%A3%85%E5%BC%80%E5%8F%91%E7%89%88%E5%9B%BA%E4%BB%B6%E5%B9%B6%E5%BC%80%E5%90%AF-ssh-%E5%8A%9F%E8%83%BD)
