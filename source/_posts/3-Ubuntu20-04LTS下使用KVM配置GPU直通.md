---
title: Ubuntu20.04LTS下使用KVM配置GPU直通
categories:
  - 笔记
tags:
  - Linux
  - KVM
abbrlink: 7780
date: 2022-09-18 00:57:11
---

查看CPU是否支持硬件虚拟化`egrep -o '(vmx|svm)' /proc/cpuinfo`

## 一、安装kvm

```shell
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
```

- `qemu-kvm` -为KVM管理程序提供硬件仿真的软件。
- `libvirt-daemon-system` -用于将libvirt守护程序作为系统服务运行的配置文件。
- `libvirt-clients` -用于管理虚拟化平台的软件。
- `bridge-utils` -一组用于配置以太网桥的命令行工具。
- `virtinst` -一组用于创建虚拟机的命令行工具。
- `virt-manager` -易于使用的GUI界面和支持命令行工具，用于通过libvirt管理虚拟机。

> 安装软件包后，libvirt守护程序将自动启动。您可以通过键入以下内容进行验证：
> 
> ```shell
> sudo systemctl is-active libvirtd
> #输出
> active
> ```
> 
> 为了能够创建和管理虚拟机，您需要将用户添加到“ libvirt”和“ kvm”组中。为此，请输入：
> 
> ```shell
> sudo usermod -aG libvirt $USER
> sudo usermod -aG kvm $USER
> ```

## 二、确认内核是否支持iommu

`cat /proc/cmdline | grep iommu`

若没有输出结果，添加`intel_iommu=on`到grub的启动参数（添加完需要重启）

`vim /etc/default/grub`	找到*GRUB_CMDLINE_LINUX_DEFAULT或GRUB_CMDLINE_LINUX*加入**intel_iommu=on**，如下：

GRUB_CMDLINE_LINUX_DEFAULT="**intel_iommu=on**"

更新grub：`sudo update-grub`

重启！

检查是否开启成功：**dmesg | grep IOMMU**，成功则显示`DMAR: IOMMU enabled`

## 三、将显卡从宿主机解绑

### 1、查看pci设备信息

`lspci -nn | grep NVIDIA`

0000:65:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107GL [Quadro P1000] [10de:1cb1] (rev a1)
0000:65:00.1 Audio device [0403]: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:0fb9] (rev a1)

### 2、查看驱动

`lspci -vv -s 65:00.0 | grep driver`

Kernel driver in use: nouveau  （ubuntu系统为显卡绑定的默认驱动）

`lspci -vv -s 65:00.1 | grep driver`

Kernel driver in user: snd_hda_intel （显卡上附带的集成声卡的默认驱动）

**禁用显卡的默认驱动**

禁用nouveau， snd_hda_intel，将其添加到`/etc/modprobe.d/blacklist.conf`文件最后

blacklist nouveau

blacklist snd_hda_intel

> ***已验证：Ubuntu20.04不需要下列步骤***
> 
> $ modprobe pci_stub
> 
> $ echo "10de 1b80" > /sys/bus/pci/drivers/pci-stub/new_id
> 
> $ echo 0000:01:00.0 > /sys/bus/pci/devices/0000:01:00.0/driver/unbind
> 
> $ echo 0000:01:00.0 > /sys/bus/pci/drivers/pci-stub/bind
> 
> $ echo "10de 10f0" > /sys/bus/pci/drivers/pci-stub/new_id
> 
> $ echo 0000:01:00.1 > /sys/bus/pci/devices/0000:01:00.1/driver/unbind
> 
> $ echo 0000:01:00.1 > /sys/bus/pci/drivers/pci-stub/bind

操作完成，重启机器

## 四、添加显卡至KVM的虚拟机

利用virt-manager操作即可

> CPU Model选择 host model，否则会报`CUDA error: all CUDA-capable devices are busy or unavailable`

安装驱动可能失败，这是因为NVIDIA不允许虚拟机使用，应该修改vm的配置文件

virsh edit `<VM Name>`

```xml
# </devices>前添加需要穿透的GPU信息：

<hostdev mode='subsystem' type='pci' managed='yes'>
  <source>
    <address domain='0x0000' bus='0x65' slot='0x00' function='0x0'/>
  </source>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x09' function='0x0'/>
</hostdev>
```

跳过显卡校验：

```xml
# 在标签</features>前添加：

<kvm>
      <hidden state='on'/>
</kvm>
```

## 五、网络配置

虚拟机网络配置需要桥接，建议使用cockpit插件cockpit-machine进行配置

## 六、安装NVIDIA驱动

{% note success simple %}
如何安装NVIDIA显卡驱动，请参考[这篇文章](https://ivmoe.github.io/posts/65062/)。
{% endnote %}