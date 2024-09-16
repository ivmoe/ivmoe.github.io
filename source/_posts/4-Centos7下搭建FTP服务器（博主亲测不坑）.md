---
title: CentOS7下搭建FTP服务器（博主亲测不坑）
categories:
  - 笔记
tags:
  - Linux
abbrlink: 52509
date: 2022-09-25 22:57:49
---

{% note success simple %}
领导要用，虽然不知道用途是干啥的，估计跟我们最近做的项目有关系。网络上搜出来的教程很多，配置复杂，而且不一定成功，很烦，我的配置肯定能用，保证不坑。
{% endnote %}

## 一、关闭防火墙、SELinux

```bash
systemctl stop firewalld.service            #关闭防火墙
systemctl disable firewalld.service         #禁止防火墙开机启动

vi /etc/selinux/config  
#永久关闭SELinux，修改SELINUX=enforcing为“SELINUX=disabled”
```

每次装完centos，第一步就要干这个，不要怀疑，Firewall还好，这个SELinux一定要关。以后的教程就不说这一步了。

## 二、安装VSFTP

以下我直接贴Linux命令了，大家注意在搭建中将其中某些目录等，更改为自己想要的。

```bash
rpm -q vsftpd    #检查vsftp是否已安装

yum -y install vsftp    #如果没有，请执行命令，如已安装请跳过

rpm -q vsftpd    #再次检查下安装信息
vsftpd-3.0.2-25.el7.x86_64

systemctl status vsftp.service    #查看运行状态，输出为下

● vsftpd.service - Vsftpd ftp daemon
   Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 四 2019-06-20 17:31:52 CST; 15h ago
  Process: 20583 ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf (code=exited, status=0/SUCCESS)
 Main PID: 20584 (vsftpd)
    Tasks: 1
   CGroup: /system.slice/vsftpd.service
           └─20584 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

6月 20 17:57:34 ftp vsftpd[21169]: pam_unix(vsftpd:auth): authentication failure; logname= uid=0 e...ogon
6月 20 17:57:54 ftp vsftpd[21183]: pam_unix(vsftpd:auth): check pass; user unknown
6月 20 17:57:54 ftp vsftpd[21183]: pam_unix(vsftpd:auth): authentication failure; logname= uid=0 e...ogon
6月 20 17:58:06 ftp vsftpd[21191]: pam_unix(vsftpd:auth): check pass; user unknown
6月 20 17:58:06 ftp vsftpd[21191]: pam_unix(vsftpd:auth): authentication failure; logname= uid=0 e...ogon
6月 20 17:58:12 ftp vsftpd[21211]: pam_unix(vsftpd:auth): check pass; user unknown
6月 20 17:58:12 ftp vsftpd[21211]: pam_unix(vsftpd:auth): authentication failure; logname= uid=0 e...ogon
6月 20 18:07:57 ftp vsftpd[21363]: pam_unix(vsftpd:auth): check pass; user unknown
6月 20 18:07:57 ftp vsftpd[21363]: pam_unix(vsftpd:auth): authentication failure; logname= uid=0 e...ogon
6月 20 18:08:51 ftp vsftpd[21392]: pam_unix(vsftpd:auth): authentication failure; logname= uid=0 e...oftp
Hint: Some lines were ellipsized, use -l to show in full.

systemctl enabled vsftp    #设置开机启动
```

## 三、配置

### 1. 备份

先备份一下配置文件，免得跟博主一样，后边服务启动失败

`cp /etc/vsftp/vsftp.conf /etc/vsftp/vsftp.conf_bak`

### 2. 编辑vsftpd.conf配置文件

```vim /etc/vsftp/vsftp.conf```

```bash
//修改以下参数
anonymous_enable=NO    //禁用匿名登录
chroot_local_user=YES    //启用限定用户在其主目录下
write_enable=YES
local_enable=YES
listen=NO    //listen改为NO，不然可能服务会启动失败

//文件末尾添加
allow_writeable_chroot=YES    //不添加此项,连接ftp服务器的时候会报500 OOR
```

其他的就不用修改了，我ftp是本地用户模式，简单省事。

### 3. 新建系统用户和用户目录

```bash
mkdir /home/ftp
useradd -d /home/ftp 用户名    #新建用户并制定用户主目录
chown -R 用户名:用户名 /home/ftp    #赋予用户相应权限
passwd 用户名    #修改用户密码
```

然后重启服务：`systemctl restart vsftp.service`

如果该用户登录不了，需在ftpusers和user_list文件中删除。
`vi /etc/vsftp/user_list`

### 4. 配置文件说明

```bash
配置的选项及说明
1、 核心设置 
 local_enable=YES    //允许本地用户登录
 write_enable=YES   //本地用户的写权限
 local_umask=022    //使用FTP的本地文件权限,默认为077，一般设置为022
 pam_service_name=vsftpd   //验证方式
 connect_from_port_20=YES  //启用FTP数据端口的数据连接
 listen=NO               // 以独立的FTP服务运行
 listen_port=23          //修改连接端口
2、匿名登录设置
 anon_upload_enable=YES         // 如果允许匿名登录，是否开启匿名上传权限
 anon_mkdir_write_enable=YES   //如果允许匿名登录，是否允许匿名建立文件夹并在文件夹内上传文件
 anon_other_write_enable=YES   // 如果允许匿名登录，匿名帐号可以有删除的权限
 anon_world_readable_only=NO    //如果允许匿名登录，匿名的下载权限，匿名为Other,可设置目录/文件属性控制
 anon_max_rate=30000            // 如果允许匿名登录，限制匿名用户传输速率,单位bite
3、限制登录  
 userlist_enable=YES      //用userlist来限制用户访问
 userlist_deny=NO         //名单中的人不允许访问
 userlist_file=/etc/vsftpd/userlist_deny.chroot     //限制名单文件放置的路径
4、限制目录
 chroot_local_user=YES    //限制所有用户都在家目录
 chroot_list_enable=YES   //调用限制在家目录的用户名单
 chroot_list_file=/etc/vsftpd/chroot_list   //限制在家目录的用户名单所在路径
5、日志设置
 xferlog_file=/var/log/vsftpd.log   //日志文件路径设置
 xferlog_std_format=YES    // 使用标准的日志格式
6、安全设置 
  idle_session_timeout=600  //用户空闲超时,单位秒
  data_connection_timeout=120   //数据连接空闲超时,单位秒
  accept_timeout=60    //将客户端空闲1分钟后断开
  local_max_rate=10000  //本地用户传输速率,单位bite
  max_clients=100      //FTP的最大连接数
  max_per_ip=3       //每IP的最大连接数
7、被动模式设置
  pasv_enable=YES     //是否开户被动模式
  pasv_min_port=3000   // 被动模式最小端口
  pasv_max_port=5000  //被动模式最大端口
```