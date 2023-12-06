---
date: 2021-11-16
authors: [darmarj]
description: >
  Proxmox v7 Subscription disable
categories:
  - Proxmox
---

# Proxmox v7 Subscription

## Usage

!!! note

    Move into **/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js** and chage as following:

__Code__

```javascript

    if (res === null || res === undefined ||!res || res.data.status.toLowerCase() !== 'active') {

    if (false) {
```
```bash

    systemctl restart pveproxy.service
    reboot
```
