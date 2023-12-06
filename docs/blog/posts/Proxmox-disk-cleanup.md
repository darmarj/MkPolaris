---
date: 2022-12-28
authors: [darmarj]
description: >
  Proxmox
categories:
  - Proxmox
---

# Proxmox disk cleanup

```bash
# use ncdu command to check the directory usage under root
nudc /
```

# Linux old kernel cleanup
```bash
OLD_KERNEL= dpkg -l | awk '{print $2}' | grep -e ^pve-.*$c

apt-get remove --purge $OLD_KERNEL
```
