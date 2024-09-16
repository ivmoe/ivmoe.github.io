---
title: Enable etcrc.local with systemd on Ubuntu 20.04
categories: 笔记
tags: Linux
abbrlink: 30648
date: 2021-11-29 13:50:00
---

{% note success simple %}
在Ubuntu20.04上实现可systemd管理的/etc/rc.local
{% endnote %}

> Certain Linux distributions that use [SystemD](https://en.wikipedia.org/wiki/Systemd)such as Ubuntu 20.04 may not allow you to run /etc/rc.local when the system is booting. In this tutorial we will go through how to enable /etc/rc.local with systemd during system boot on Ubuntu 20.04.

### Procedure to setup /etc/rc.local with systemd on Ubuntu 20.04

Check the current status of rc-local service

```
sudo systemctl status rc-local
```

### Enable rc.local service

Enable /etc/rc.local to run on system boot using the command

```
sudo systemctl enable rc-local
```

As you may have already read, it is not possible to enable rc.local at startup
using SystemD on Ubuntu 20.04. Therefore we have to do this another way.

### Manually create a systemd service

We will need to manually create a SystemD service which will start at system
boot.

```
sudo vi /etc/systemd/system/rc-local.service
```

Now enter the following text, save and close the file.

```
[Unit]
 Description=/etc/rc.local Compatibility
 ConditionPathExists=/etc/rc.local

[Service]
 Type=forking
 ExecStart=/etc/rc.local start
 TimeoutSec=0
 StandardOutput=tty
 RemainAfterExit=yes
 SysVStartPriority=99

[Install]
 WantedBy=multi-user.target
```

### Create and Edit rc.local file

Now we will need to edit the /etc/rc.local file. Issue the following command
and press Enter

```
sudo vi /etc/rc.local
```

Paste in the following, this ensures that the script is bash executable, all
bash scripts shoul have this at the top

```
#!/bin/bash
```

save and close the file.

We will now need to append permissions to make the newly created file
executable. Issue the following command and press Enter

```
sudo chmod +x /etc/rc.local
```

### Enable the service on boot

enable rc.local with systemd on Ubuntu 20.04

After that, enable the service on system boot

```
sudo systemctl enable rc-local
```

Now let’s reboot the system and check if the rc-local SystemD service is
working.

When your system has booted up again, issue the following command

```
sudo systemctl status rc-local
```

Enabling /etc/rc.local with systemd is completed on Ubuntu 20.04 startup.

转载自：https://marsown.com/wordpress/how-to-enable-etc-rc-local-with-systemd-on-ubuntu-20-04/