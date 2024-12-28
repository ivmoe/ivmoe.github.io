---
title: Use Docker-Compose to run Mysql8.0 and Redis
categories:
  - 笔记
tags:
  - Docker
abbrlink: 13078
date: 2024-11-05 18:35:26
updated: 2024-11-12 15:35:10
sticky: 
---

> This article tell you how to use docker-compose to run Mysql8.0 and Redis.
>
> And I will not tell you how to install Docker-ce. You can use search engine like: Bing or Google.



# Docker registry-mirrors

If you are in China, you should use proxy to pull images from DockerHub.

You can use my config, the file is `/etc/docker/daemon.json`:

```json
{
	"registry-mirrors": [
	    "https://docker.1panel.dev",
	    "https://dockerproxy.net"
	]
}
```

# Create directory and files

1. Create `docker-compose.yml`

   ```yaml
   # version: '3.8'  # Now the version attribute is obsoleted in latest version.
   services:
     mysql:
       image: mysql:8.0
       restart: always
       container_name: mysql-db
       ports:
         - 3306:3306
       logging:
         driver: 'json-file'
         options:
           max-size: '1g'
       environment:
         # - TZ=Asia/Shanghai
         # - LANG=en_US.UTF-8
         # - MYSQL_ROOT_HOST='%'
         # 配置root密码
         - MYSQL_ROOT_PASSWORD=123456
       volumes:
         # 挂载数据目录
         - ./mysql/data:/var/lib/mysql
         # 挂载配置文件目录
         - ./mysql/my.cnf:/etc/mysql/my.cnf
         - /etc/localtime:/etc/localtime:ro
   
     redis:
       image: redis:7.2.5-alpine
       restart: always
       container_name: Redis
       ports:
         - "6379:6379"
       command: redis-server /etc/redis/redis.conf
       volumes: 
         - ./redis/redis.conf:/etc/redis/redis.conf
         - ./redis/logs:/logs
         - ./redis/data:/data
   ```

2. Create `redis.conf`
	```yaml
    # 绑定监听IP地址
    bind 0.0.0.0
    # 自定义密码
    # requirepass root		# 若需要登录密码可在这里设置密码并取消注释
    # 启动端口
    port 6379
    # redis 默认就开启 rdb 全量备份，以下是默认的备份触发机制
    # 900s内至少一次写操作则执行bgsave进行RDB持久化
    save 900 1
    save 300 10
    save 60 10000
    # 是否压缩 rdb 备份文件，默认是压缩
    # 如果 redis 承载的数据量非常大的话，建议不要压缩
    # 因为压缩过程中需要耗费大量 cpu 和内存资源，磁盘相对而言比较廉价
    rdbcompression yes
    # rdb 备份的文件名
    dbfilename dump.rdb
    # Redis 备份文件存储目录，注意：该路径是 docker 容器内的路径
    dir /data
    # 是否开启 aof 增量备份功能。会有一定的性能损耗，如果Redis作为cache类应用，不建议开启。
    appendonly yes
    # AOF文件的名称，这里使用默认值
    appendfilename appendonly.aof
    # aof 增量备份的策略，这里是每秒钟一次，将累积的写命令持久化到硬盘中
    appendfsync everysec
   ```
3. Create `my.cnf`
    ```yaml
    [mysqld]
    user=mysql
    default-storage-engine=INNODB
    character-set-server=utf8
    secure-file-priv=NULL # mysql 8 新增这行配置
    default-authentication-plugin=mysql_native_password  # mysql 8 新增这行配置
    
    port = 3306 # 端口与docker-compose里映射端口保持一致
    #bind-address= localhost #一定要注释掉，mysql所在容器和django所在容器不同IP
    
    basedir = /usr
    datadir = /var/lib/mysql
    tmpdir = /tmp
    pid-file = /var/run/mysqld/mysqld.pid
    socket = /var/run/mysqld/mysqld.sock
    skip-name-resolve  # 这个参数是禁止域名解析的，远程访问推荐开启skip_name_resolve。
    [client]
    port = 3306
    default-character-set=utf8
    [mysql]
    no-auto-rehash
    default-character-set=utf8
    ```

# Run Containers

```bash
sudo docker-compose up -d	# 创建启动容器并保持在后台运行

sudo docker-compose ps	# 查看以docker-compose启动的容器状态


# 以下不要执行，属于小扩展
sudo docker-compose down	# 删除容器、容器网络等
sudo docker-compose down -v	# 在以上的基础上，删除卷
```

The status of running:

```bash
a@@kelvyn:~/Desktop/db_docker$ sudo docker-compose ps
NAME       IMAGE                COMMAND                   SERVICE   CREATED          STATUS          PORTS
Redis      redis:7.2.5-alpine   "docker-entrypoint.s…"   redis     54 minutes ago   Up 54 minutes   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp
mysql-db   mysql:8.0            "docker-entrypoint.s…"   mysql     54 minutes ago   Up 54 minutes   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp
```



以上，都是单节点快速部署的方法，可用于测试、开发用途，生产勿用！后续看情况可能会出哨兵、集群模式的使用方法。
