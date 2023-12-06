---
date: 2022-03-10
authors: [darmarj]
description: >
  Terraform + KVM
categories:
  - Terraform
---

# Terraform works on KVM for IaaS

[Terraform]((https://www.terraform.io/)) is an open-source infrastructure as code software tool that provides a consistent CLI workflow to manage hundreds of cloud services. Terraform codifies cloud APIs into declarative configuration files.

## Summary

!!! failure "PopOS"
    **libvirt_domain.domain: Error creating libvirt domain**: virError(Code=1, Domain=10, Message='internal error: process exited while connecting to monitor: 2019-01-28T02:29:14.861688Z qemu-system-x86_64: -drive file=/var/lib/libvirt/images/volume-0,format=qcow2,if=none,id=drive-virtio-disk0: Could not open '/var/lib/libvirt/images/volume-0': **Permission denied')**

    {++The reason on the problem++}:

    {++On Ubuntu/Debian distros **SELinux** is enforced by qemu even if it is disabled globally, this might cause unexpected++}

[**Walkaround**](https://lifesaver.codes/answer/permission-denied-546):

```bash
#/etc/libvirt/qemu.conf
~ security_driver = "none"
~ sudo systemctl restart libvirtd.service
```
