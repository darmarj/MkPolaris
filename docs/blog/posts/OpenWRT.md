---
date: 2023-05-25
authors: [darmarj]
description: >
  OpenWRT
categories:
  - OpenWRT
---

# OpenWRT
## Parition switch

``` bash
root@lede:~# fw_printenv boot_part
boot_part=1
```

``` bash
fw_setenv boot_part 2
reboot
```
