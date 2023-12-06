---
date: 2022-03-26
authors: [darmarj]
description: >
  KVM Bridge
categories:
  - KVM
---

# KVM Bridge

## Linux

Use `nmtui` command to config the **Bridge** adapter on-premise.

``` bash
NAME
nmtui - Text User Interface for controlling NetworkManager
```

Click "Edit a connection" option to start walking through.
![nmtui](../../../assets/images/nmtui.png "nmtui")

Open the page once click "Add" button for "Bridge" creation

![EditConnectionForBridge](../../../assets/images/EditConnectionForBridge.png "EditConnectionForBridge")

![SlaveConnection](../../../assets/images/SlaveConnection.png "SlaveConnection")

!!! Tips
    Leave the empty on the slave device for communication automatically by adapter and router.

Delete the default wired connection and waiting a while. New **Bridge** connection will be activated soon later.

``` bash
btctl show

bridge name     bridge id               STP enabled     interfaces
nm-bridge       ID                      status          adapterName
```
