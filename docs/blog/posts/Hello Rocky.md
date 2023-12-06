---
date: 2022-01-25
authors: [darmarj]
description: >
  RockyLinux system on how to setup and Docker container installation
categories:
  - RockyLinux
---

# Rocky Linux

__[Rocky Linux](https://rockylinux.org/) is an open-source enterprise operating system designed to be 100% bug-for-bug compatible with Red Hat Enterprise Linux®. It is under intensive development by the community.__

## [Mirror Resources](https://mirror.rockylinux.org/mirrormanager/mirrors)
| Country     |             Mirror Name             |    Categories      | Bandwidth |
| ----------- | ----------------------------------- | ------------------ | --------- |
|   __JP__    |                JAIST                | ftp.jaist.ac.jp    | 60000     |
|   __CN__    | eScience Center, Nanjing University | mirrors.nju.edu.cn | 10000     |
|   __SG__    |               RPMDB.org             | sg.rpmdb.org       | 10000     |

``` bash title="Replace Method"
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://ftp.jaist.ac.jp/rocky|g'
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo

dnf makecache
```

``` bash title="Revert Method"
sed -e 's|^#mirrorlist=|mirrorlist=|g' \
    -e 's|^baseurl=https://mirrors.sjtug.sjtu.edu.cn/rocky|#baseurl=http://dl.rockylinux.org/$contentdir|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo

dnf makecache
```

### Docker
``` shell title="docker repo & service control"
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo && dnf update
dnf install -y docker-ce docker-ce-cli containerd.io
systemctl enable --now docker
systemctl status docker
```
!!! tip "KMS Service"
    ``` shell
    docker pull mikolatero/vlmcsd   # updated last year ago
    docker run -d -p 1688:1688 --restart=always --name vlmcsd mikolatero/vlmcsd

    docker pull mogeko/vlmcsd       # the latest replacement
    docker run -d --name vlmcsd -p 1688:1688 --restart unless-stopped ghcr.io/mogeko/vlmcsd:latestj
    ```

### Docker-Compose
``` shell title="docker-compose installation"
e.g. #VERSION = 1.29.2
sudo curl -L "https://github.com/docker/compose/releases/download/#VERSION/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
``` markdown title="Text with annotations"
Lorem ipsum dolor sit amet, (1) consectetur adipiscing elit.
{ .annotate }

1.  :man_raising_hand: I'm an annotation! I can contain `code`, __formatted
    text__, images, ... basically anything that can be expressed in Markdown.
```
