---
title: >-
  KVM虚拟机启动报错“unable to set AppArmor profile 'libvirt-' for
  'usrbinqemu-system-x86_64' No such file or directory”
categories: 笔记
tags: KVM
abbrlink: 1253
date: 2021-07-15 15:22:00
---

我的虚拟机稳定的运行了一段时间，由于参加WAIC展会，期间晚上断电关机，所以宿主机需要较为频繁的重启。

这事在展会的第四天终于出问题了。我一共是5个虚拟机，早上开机，稳稳的起来了三个，但是有两个挺重要的虚机没有起来，报错如下：

{% note danger simple %}
internal error: Process exited prior to exec: libvirt:  error : unable to set AppArmor profile 'libvirt-91916cbc-63e6-437c-9932-4ce813b079d2' for '/usr/bin/qemu-system-x86_64': No such file or directory
{% endnote %}

解决办法：

```bash
sudo rm /etc/apparmor.d/libvirt/libvirt*
```