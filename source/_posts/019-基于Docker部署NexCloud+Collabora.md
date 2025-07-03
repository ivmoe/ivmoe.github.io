---
title: 基于 Docker 部署 NexCloud + Collabora
categories:
  - 笔记
tags:
  - Docker
  - NextCloud
abbrlink: 100019
date: 2025-7-3 12:26:45
updated: 
sticky: 
---

# 二进制安装 Docker

Docker 下载地址：[Download](https://download.docker.com/linux/static/stable/)
Docker Compose 下载地址：https://github.com/docker/compose/releases

```bash
wget https://download.docker.com/linux/static/stable/x86_64/docker-28.3.1.tgz -O docker-28.3.1.tgz

tar  xzf docker-*.tgz
mv docker/* /usr/bin/

# cat > /etc/docker/daemon.json << EOF
# {
#     "data-root": "${DOCKER_ROOT_DIR}"
# }
# EOF

cat > /etc/systemd/system/docker.service << EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutStartSec=0
RestartSec=2
Restart=always

LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target
EOF
chmod 755  /etc/systemd/system/docker.service
systemctl daemon-reload
systemctl enable docker.service --now

groupadd docker
usermod -aG docker $USER


chmod +x docker-compose*
cp docker-compose* /usr/bin/docker-compose
```


# 创建 Docker Compose 文件

```yaml
services:
  mysql:
    container_name: mysql
    image: mysql:8.0.32
    restart: always
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=Kelvyn123
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=Kelvyn123
      - TZ=Asia/Shanghai
    volumes:
      - ./db:/var/lib/mysql
    networks:
      nextcloud:
        ipv4_address: 172.23.10.12

  collabora:
    container_name: collabora
    privileged: true
    image: collabora/code:24.04.9.1.1
    restart: always
    ports:
      - "9980:9980"
    environment:
      - TZ=Asia/Shanghai
      # 这里是要修改为宿主机IP的
      - server_name=192.168.8.12:9980
    networks:
      nextcloud:
        ipv4_address: 172.23.10.13

  linux-nextcloud:
    container_name: linux-nextcloud
    image: linuxserver/nextcloud:31.0.0
    restart: unless-stopped
    depends_on:
      - mysql
      - collabora
    ports:
      - "443:443"
    environment:
      - TZ=Asia/Shanghai
      - PUID=1000
      - PGID=1000
    volumes:
      - ./nextcloud/config:/config
      - ./nextcloud/data:/data
    networks:
      nextcloud:
        ipv4_address: 172.23.10.10

networks:
  nextcloud:
    driver: bridge
    ipam:
      config:
        - subnet: 172.23.10.0/24
```
