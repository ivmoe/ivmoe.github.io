---
title: Ubuntu安装mysql并远程访问
categories:
  - 笔记
tags:
  - Linux
  - MySQL
abbrlink: 21274
date: 2023-02-19 09:23:39
---

打开mysql社区网站：https://dev.mysql.com/downloads/

由于我使用的是ubuntu，所以下载的apt的包：https://dev.mysql.com/downloads/repo/apt/

上传至服务器，运行命令：

```bash
sudo dpkg -i mysql-apt-config_0.8.24-1_all.deb
```

按照提示选择好要安装的版本，这里就是MySQL8。

```bash
sudo apt update
sudo apt install mysql-server -y

#安装好以后简单设置下，远程访问它

mysql -uroot -p  #登录mysql

2.use mysql;  #切换mysql数据库

3.update user set host = '%' where user = 'root';  #更新用户权限

4.flush privileges;  #刷新权限

5.select user,host from user;  #检查root是否对应%
```