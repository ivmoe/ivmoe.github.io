---
title: Linux下安装NFS服务器
tags: Linux
abbrlink: 13583
date: 2022-08-16 21:19:29
categories: 笔记
---
> 安装Linux系统，这里是CentOS7，操作全程在root下进行，关闭防火墙和SELinux。
>

## 添加新的硬盘

```shell
fdisk -l    #查看硬盘状态

Disk /dev/sdb: 3298.5 GB, 3298534883328 bytes, 6442450944 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


fdisk /dev/sdb    #对磁盘进行操作

Command (m for help): m				# 根据提示添加新的分区
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)


操作完成后按“w”保存退出。

#对分区进行格式化,此处为xfs格式，还有ext4、ext3等
mkfs -t xfs /dev/sdb1
```

创建将要作为NFS分享的文件夹：`mkdir /nfs`，更改归属组与用户`chown -R root.root /nfs`

挂载硬盘：`mount /dev/sdb1 /nfs`

永久保存挂载点

```shell
vi /etc/fstab
#填入下面的内容
/dev/sdb1       /nfs    xfs    defaults        0 0

#保存退出，重启机器
```

## 安装NFS服务

```shell
yum install -y nfs-utils rpcbind

#设置开机自启
systemctl enable rpcbind
systemctl enable nfs-server

# 编辑exports
$ vi /etc/exports

# 输入以下内容(格式：NFS共享的目录 NFS客户端地址1(参数1,参数2,...) 客户端地址2(参数1,参数2,...))
$ /nfs 192.168.2.0/24(rw,async,no_root_squash)

#如果设置为 /nfs *(rw,async,no_root_squash) 则对所有的IP都有效
```

- 常用选项：
  - ro：客户端挂载后，其权限为只读，默认选项；
  - rw:读写权限；
  - sync：同时将数据写入到内存与硬盘中；
  - async：异步，优先将数据保存到内存，然后再写入硬盘；
  - Secure：要求请求源的端口小于1024
- 用户映射：
  - root_squash:当NFS客户端使用root用户访问时，映射到NFS服务器的匿名用户；
  - no_root_squash:当NFS客户端使用root用户访问时，映射到NFS服务器的root用户；
  - all_squash:全部用户都映射为服务器端的匿名用户；
  - anonuid=UID：将客户端登录用户映射为此处指定的用户uid；
  - anongid=GID：将客户端登录用户映射为此处指定的用户gid

```shell
# 查看nfs服务端信息
$ nfsstat -s

# 查看nfs客户端信息
$ nfsstat -c
```