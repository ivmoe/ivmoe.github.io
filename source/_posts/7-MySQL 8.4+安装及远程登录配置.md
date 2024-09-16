---
title: MySQL8.4+安装及远程登录配置
categories:
  - 笔记
tags:
  - Linux
  - MySQL
date: 2024-9-16 20:53:14
---

## RockyLinux 9.4下安装
下载地址：[MySQL Community Downloads](https://dev.mysql.com/downloads/)

我下载的是<font style="color:rgb(85, 85, 85);">MySQL Yum Repository</font><font style="color:rgb(85, 85, 85);">。</font>

```bash
# 安装MySQL软件源
rpm -ivh mysql84-community-release-el9-1.noarch.rpm

# 更新软件包列表缓存
dnf makecache

# 安装MySQL
dnf install -y mysql-community-server-8.4.2-1.el9.x86_64

# 开机自启
systemctl enable mysqld.service

# 防火墙放行端口3306
firewall-cmd --permanent --zone=public --add-port=3306/tcp
# 重启防火墙服务
systemctl restart firewalld.service

```

my.cnf 文件简单配置【文件路径 `/etc/my.cnf`】

```yaml
[mysqld]

bind-address=0.0.0.0

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# 设置字符集
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# 设置默认存储引擎
default-storage-engine=INNODB

# 激活插件
mysql_native_password=ON

```

```bash
# 配置完成启动MySQL
systemctl start mysqld.service

# 查看状态
[root@kelvyn ~]# systemctl status mysqld.service
● mysqld.service - MySQL Server
     Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; preset: disabled)
     Active: active (running) since Fri 2024-09-13 19:13:13 CST; 10min ago
       Docs: man:mysqld(8)
             http://dev.mysql.com/doc/refman/en/using-systemd.html
   Main PID: 1368 (mysqld)
     Status: "Server is operational"
      Tasks: 34 (limit: 23036)
     Memory: 520.9M
        CPU: 2.634s
     CGroup: /system.slice/mysqld.service
             └─1368 /usr/sbin/mysqld

Sep 13 19:13:07 kelvyn systemd[1]: Starting MySQL Server...
Sep 13 19:13:13 kelvyn systemd[1]: Started MySQL Server.

```

## 配置MySQL远程登录
```bash
# 1. 获取MySQL的root密码
grep "password is generated" /var/log/mysqld.log | awk '{print $NF}'

# 2. 登录MySQL
mysql -uroot -p

# 3. 第一次进入数据库只能修改密码，不能做任何事
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Meng.Kelvyn123';

# 4. 关闭密码复杂度要求（可选）
show global variables like '%validate_password%';
set global validate_password.policy=0;       # 关闭密码复杂性策略
set global validate_password.length=4;      # 设置密码最低长度为4

# 5. 再次设置一个简单点的密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

USE mysql;
update user set host = '%' where user = 'root';

flush privileges; 
```

这次通过Navicatd的软件就可以直接连接了。