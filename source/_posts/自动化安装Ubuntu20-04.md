---
title: 自动化安装Ubuntu20.04
categories:
  - 笔记
tags:
  - Linux
abbrlink: 3875
date: 2022-08-25 22:12:04
updated: 2022-08-25 22:12:04
---

# 自动化安装Ubuntu20.04

{% note danger simple %}
只能使用 live server 版！！！
{% endnote %}

下载想要版本的 iso 文件备用，可在[这里](https://old-releases.ubuntu.com/releases/)下载到历史版本。

准备一个安装好的操作系统 Ubuntu20.04，将刚刚下载的 iso 文件放到操作系统中。

使用工具一键制作，在 GitHub 下载 [ubuntu-autoinstall-generator](https://github.com/covertsh/ubuntu-autoinstall-generator)。

```bash
# 要先修改下这个脚本中的验证，不然不会通过老版本的，只会有最新版本的

bash ubuntu-autoinstall-generator.sh -a -u user-data.example -s 官方iso文件 -d ubuntu-autoinstall.iso
```

user-data 这个文件怎么配置可以在 ubuntu [官方文档](https://ubuntu.com/server/docs/install/autoinstall-reference)中查看。

密码加密的命令 :

```bash
printf '密码' | openssl passwd -6 -salt 'FhcddHFVZ7ABA4Gi加密代码' -stdin
```

```bash
mkdir   /mysoft
cp -rf /var/cache/apt/archives /mysoft
chmod -R 777 /mysoft
cd /mysoft
apt-ftparchive packages archives >archives/Packages
cd archives/
gzip -c Packages > Packages.gz
touch release
apt-ftparchive release ./ >Release
修改配置文件
root@pc01:~# cp sources.list{,.bak}
root@pc01:~# vim /etc/apt/sources.list
deb [trusted=yes] file:/mysoft archives/ 
root@pc01:~# apt-get update



#######################说明##################################
后期再有包加入的时候，在执行一遍这几步就行了
apt-ftparchive packages archives >archives/Packages
gzip -c Packages > Packages.gz
apt-ftparchive release ./ >Release
###########################################################
```
