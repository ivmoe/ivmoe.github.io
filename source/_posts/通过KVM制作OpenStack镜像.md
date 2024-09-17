---
title: 通过KVM制作OpenStack镜像
categories: 
    - 笔记
tags: 
    - KVM
    - OpenStack
abbrlink: 19535
date: 2022-08-21 14:08:35
updated: 2022-08-21 14:08:35
---

## 在Ubuntu20.04上安装KVM

运行以下命令以安装KVM和其他虚拟化管理软件包：

```bash
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
```

- `qemu-kvm` -为KVM管理程序提供硬件仿真的软件。
- `libvirt-daemon-system` -用于将libvirt守护程序作为系统服务运行的配置文件。
- `libvirt-clients` -用于管理虚拟化平台的软件。
- `bridge-utils` -一组用于配置以太网桥的命令行工具。
- `virtinst` -一组用于创建虚拟机的命令行工具。
- `virt-manager` -易于使用的GUI界面和支持命令行工具，用于通过libvirt管理虚拟机。

安装软件包后，libvirt守护程序将自动启动。您可以通过键入以下内容进行验证：

```bash
sudo systemctl is-active libvirtd

#输出
active
```

为了能够创建和管理虚拟机，您需要将用户添加到“ libvirt”和“ kvm”组中。为此，请输入：

```bash
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```

`$USER` 是一个环境变量，其中包含当前登录用户的名称。

注销并重新登录，以便刷新组成员身份。

## 压缩qcow2镜像文件

虚拟机需要关机，qcow2的存放位置：`/var/lib/libvirt/images/`

```bash
#复制镜像文件到其他目录
sudo cp /var/lib/libvirt/images/tes.qcow2 /data/test.qcow2

#导出压缩后的镜像文件
sudo qemu-img convert -c -O qcow2 test.qcow2 new.qcow2
```