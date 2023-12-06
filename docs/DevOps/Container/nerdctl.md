# Nerdctl

**Nerdctl: Docker-compatible CLI for containerd**

## Why another CLI?
containerd already has its own CLI called ctr . However, ctr was made only for work very low-level functionality of containerd, and hence its CLI design is not friendly to humans. So we had to create another CLI with high-level functionalities and with human-friendly UI/UX.
Notably, ctr lacks the equivalents of the following Docker CLI commands:

- docker run -p <PORT>
- docker run --restart=always
- docker pull with ~/.docker/config.json and credential helper binaries such as docker-credential-ecr-login
- docker logs

## Getting started with nerdctl

The latest binary release of nerdctl can be [downloaded](https://github.com/containerd/nerdctl/releases).

Two types of distributions are available:

- **nerdctl-<VERSION>-linux-amd64.tar.gz** : nerdctl only. Should be extracted under /usr/local/bin .
- **nerdctl-full-<VERSION>-linux-amd64.tar.gz** : nerdctl with dependencies (containerd, runc, CNI, …). Should be extracted under /usr/local .

If you already have containerd, you should use the former one. Otherwise the latter one is the best choice.

### :whale: **Installation**
``` bash
➜ wget https://download.fastgit.org/containerd/nerdctl/releases/download/v0.12.1/nerdctl-0.12.1-linux-amd64.tar.gz
➜ mkdir -p /usr/local/containerd/bin/ && tar -zxvf nerdctl-0.12.1-linux-amd64.tar.gz nerdctl && mv nerdctl /usr/local/containerd/bin/
➜ ln -s /usr/local/containerd/bin/nerdctl /usr/local/bin/nerdctl

```
```
➜ nerdctl run -d -p 80:80 --name=nginx --restart=always nginx:alpine
➜ nerdctl exec -it nginx /bin/sh
/ # date
Thu Aug 19 06:43:19 UTC 2021
/ #
```

### :whale: **Nerdctl**

1. **nerdctl inspect**: Return low-level information on objects.

2. **nerdctl tag**: Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
``` bash
➜ nerdctl images
REPOSITORY    TAG                  IMAGE ID        CREATED           SIZE
busybox       latest               0f354ec1728d    6 minutes ago     1.3 MiB
nginx         alpine               bead42240255    41 minutes ago    16.0 KiB
➜ nerdctl tag nginx:alpine harbor.k8s.local/course/nginx:alpine
➜ nerdctl images
REPOSITORY                       TAG                  IMAGE ID        CREATED           SIZE
busybox                          latest               0f354ec1728d    7 minutes ago     1.3 MiB
nginx                            alpine               bead42240255    41 minutes ago    16.0 KiB
harbor.k8s.local/course/nginx    alpine               bead42240255    2 seconds ago     16.0 KiB
```

3. **nerdctl save**: Save one or more images to a tar archive (streamed to STDOUT by default)
``` bash
➜ nerdctl save -o busybox.tar.gz busybox:latest
➜ ls -lh busybox.tar.gz
-rw-r--r-- 1 root root 761K Aug 19 15:19 busybox.tar.gz
```
4. **nerdctl load**: Load an image from a tar archive or STDIN
``` bash
➜ nerdctl load -i busybox.tar.gz
unpacking docker.io/library/busybox:latest (sha256:0f354ec1728d9ff32edcd7d1b8bbdfc798277ad36120dc3dc683be44524c8b60)...done
```

5. **nerdctl build**: Build an image from a Dockerfile. Needs buildkitd to be running.
!!! tip
	BuildKit is turns **a Dockerfile into a Docker image**. And it doesn’t just build Docker images; it can build **OCI images** and **several other output formats**.

	BuildKit is composed of the **buildkitd daemon** and the **buildctl client**.
	While the buildctl client is available for Linux, macOS, and Windows, the buildkitd daemon is only available for Linux currently.

	The buildkitd daemon requires the following components to be installed:

    - runc or crun
    - containerd (if you want to use containerd worker)


	[Github](https://github.com/moby/buildkit)

``` bash
➜ nerdctl build -t nginx:nerdctl -f Dockerfile .
FATA[0000] `buildctl` needs to be installed and `buildkitd` needs to be running, see https://github.com/moby/buildkit: exec: "buildctl": executable file not found in $PATH
```

:octicons-package-dependents-16: [**BuildKit**](https://github.com/moby/buildkit)
``` bash
➜ wget https://github.com/moby/buildkit/releases/download/v0.9.1/buildkit-v0.9.1.linux-amd64.tar.gz
# download.fastgit.org for speedup
# wget https://download.fastgit.org/moby/buildkit/releases/download/v0.9.1/buildkit-v0.9.1.linux-amd64.tar.gz
➜ tar -zxvf buildkit-v0.9.1.linux-amd64.tar.gz -C /usr/local/containerd/
bin/
bin/buildctl
bin/buildkit-qemu-aarch64
bin/buildkit-qemu-arm
bin/buildkit-qemu-i386
bin/buildkit-qemu-mips64
bin/buildkit-qemu-mips64el
bin/buildkit-qemu-ppc64le
bin/buildkit-qemu-riscv64
bin/buildkit-qemu-s390x
bin/buildkit-runc
bin/buildkitd
➜ ln -s /usr/local/containerd/bin/buildkitd /usr/local/bin/buildkitd
➜ ln -s /usr/local/containerd/bin/buildctl /usr/local/bin/buildctl
```

:octicons-plug-16: Systemd
``` bash
➜ cat /etc/systemd/system/buildkit.service
[Unit]
Description=BuildKit
Documentation=https://github.com/moby/buildkit

[Service]
ExecStart=/usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true

[Install]
WantedBy=multi-user.target
```

``` bash
➜ systemctl daemon-reload
➜ systemctl enable buildkit --now
Created symlink /etc/systemd/system/multi-user.target.wants/buildkit.service → /etc/systemd/system/buildkit.service.
➜ systemctl status buildkit
● buildkit.service - BuildKit
     Loaded: loaded (/etc/systemd/system/buildkit.service; enabled; vendor preset: enabled)
     Memory: 8.6M
     CGroup: /system.slice/buildkit.service
             └─5779 /usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true

Aug 19 16:03:10 ydzsio systemd[1]: Started BuildKit.
Aug 19 16:03:10 ydzsio buildkitd[5779]: time="2021-08-19T16:03:10+08:00" level=warning msg="using host network as the default"
Aug 19 16:03:10 ydzsio buildkitd[5779]: time="2021-08-19T16:03:10+08:00" level=info msg="found worker \"euznuelxhxb689bc5of7pxmbc\", labels>
Aug 19 16:03:10 ydzsio buildkitd[5779]: time="2021-08-19T16:03:10+08:00" level=info msg="found 1 workers, default=\"euznuelxhxb689bc5of7pxm>
Aug 19 16:03:10 ydzsio buildkitd[5779]: time="2021-08-19T16:03:10+08:00" level=warning msg="currently, only the default worker can be used."
Aug 19 16:03:10 ydzsio buildkitd[5779]: time="2021-08-19T16:03:10+08:00" level=info msg="running server on /run/buildkit/buildkitd.sock"
~
```

### :whale: **Nerdctl Compose**

1. **nerdctl compose up**: Create and start containers
2. **nerdctl compose build**: Build or rebuild services
3. **nerdctl compose config**: Validate and view the Compose file
