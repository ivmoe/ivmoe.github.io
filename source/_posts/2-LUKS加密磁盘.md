---
title: LUKS加密磁盘
categories:
  - 笔记
tags:
  - Linux
  - LUKS
abbrlink: 23942
date: 2022-08-28 19:28:54
---

> 软件包 cryptsetup

## 加密分区

创建分区后，加密分区

```bash
cryptsetup luksFormat [Device]/dev/sda2

例：cryptsetup luksFormat /dev/sda2
```

输入大写的“YES”，回车

输入两次密码，密码复杂度有要求

## 打开加密分区

输入命令，并创建加密分区的别名

```bash
cryptsetup luksOpen [Device] [Name]

[Name]是指加密后的分区名称，在/dev/mapper中体现

例：cryptsetup luksOpen /dev/sda2 cr_data
```

## 挂载加密分区

```bash
mkfs.ext4 /dev/mapper/[Name]

例：mkfs.ext4 /dev/mapper/cr_data

mount /dev/mapper/cr_data /data
```

## 自动解密挂载

```bash
#添加解密文件
cryptsetup luksAddKey /dev/sda2 /root/cryptpasswd
#/root/cryptpasswd 文件中写入新的密码

echo "Ivmoe123" | cryptsetup luksAddKey /dev/sda2 /root/cryptpasswd

vim /etc/crypttab
写入内容：
  cr_data  /dev/sda2  /root/cryptpasswd
	建议 /root/cryptpasswd 使用随机数
	#!/bin/bash
	spasswd=$(openssl rand -base64 10)
	echo $spasswd > /root/cryptpasswd

vim /etc/fstab
写入内容：
	cr_data  /data  ext4  defaults  0  0
```