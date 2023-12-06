---
date: 2021-11-21
authors: [darmarj]
description: >
  Proxmox v7 source in PandaWorld
categories:
  - Proxmox
---



# Proxmox v8 Source in PandaWorld

## Usage

!!! note

    comment the context for disable the _**ENTERPRISE**_ repo in following file
    ```shell
    nano /etc/apt/sources.list.d/pve-enterprise.list
    ```

### Domestic ATP resource of ustc

```shell
# GPG key sync up
wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
```
```shell
# The apt resource for no-subscription
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```
```shell
# Replace the ustc mirrors on latest Proxmox Debian system
sed -i 's|^deb http://ftp.debian.org|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sed -i 's|^deb http://security.debian.org|deb https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
```
```shell
# Replace the Ceph resource
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list
```
```shell
# Replace the CT(Container) resource
sed -i 's|http://download.proxmox.com|https://mirrors.ustc.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```

# Proxmox v7 Source in PandaWorld

## Usage

!!! note

    Moving into **/etc/apt/sources.list.d/pve-enterprise.list** and comments the original enterprise sources as below:

```markdown
echo "#deb https://enterprise.proxmox.com/debian/pve bullseye pve-enterprise"
> /etc/apt/sources.list.d/pve-enterprise.list
```

### Domestic APT resource

    vi /etc/apt/sources.list

```markdown
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
```
