---
title: Ubuntu18.04安装Nvidia显卡驱动
categories: 笔记
tags: Linux
abbrlink: 65062
date: 2020-11-20 15:44:00
---
# 下载显卡驱动

`lspci | grep -i nvidia` 查看显卡型号。

从Nvidia官网下载相应驱动：[https://www.nvidia.com/Download/index.aspx](https://www.nvidia.com/Download/index.aspx)

# 关闭图形界面

Ubuntu18.04已经与之前的版本不一样了，比较麻烦。

```sbash
# Ubuntu18.04
    #关闭图形界面
$ sudo systemctl set-default multi-user.target
$ sudo reboot
    #开启图形界面
$ sudo systemctl set-default graphical.target
$ sudo reboot

# Ubuntu16.04
sudo service lightdm stop
sudo service lightdm start
```

# 禁用nouveau

Ubuntu 系统默认安装好是使用的一个开源的驱动： `nouveau`，我们要安装官方的驱动需要先禁用这个开源驱动，方法如下，依次执行：

```bash
sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
```

查看是否禁用成功：

```bash
$ cat /etc/modprobe.d/blacklist-nvidia-nouveau.conf
blacklist nouveau
options nouveau modeset=0           # 表示成功

$ sudo update-initramfs -u  # 重新加载驱动

$ sudo reboot   ##重启生效

$ lsmod | grep nouveau   # 无任何输出即表示成功
```

# 安装Nvidia驱动

```bash
# 安装依赖
$ sudo apt install gcc g++ make dkms
#提升驱动安装脚本的执行权限
$ sudo chmod +x NVIDIA-Linux-x86_64-384.98.run #根据下载好的版本输入
$ sudo ./NVIDIA-Linux-x86_64-384.98.run
# 按照提示选择安装即可
# 安装完成后测试是否安装成功
$ nvidia-smi

# 输出如下即表示安装成功
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 384.98                 Driver Version: 384.98                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GT 635M     Off  | 00000000:01:00.0 N/A |                  N/A |
| N/A   47C    P0    N/A /  N/A |      0MiB /  1985MiB |     N/A      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0                    Not Supported                                       |
+-----------------------------------------------------------------------------+
```