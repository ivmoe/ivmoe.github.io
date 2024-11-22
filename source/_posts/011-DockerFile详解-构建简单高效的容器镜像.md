---
title: DockerFile详解-构建简单高效的容器镜像
categories:
  - 笔记
tags:
  - docker
abbrlink: 
date: 2024-11-22 09:41:54
updated: 
sticky: 
---

## Dockerfile简介

Dockerfile是一个包含了一系列命令的文本文件，这些命令可以用于自动化地创建一个Docker镜像。**通过编写Dockerfile，可以将环境配置、应用程序代码、依赖关系等打包成一个镜像，便于快速创建容器。**

用户可以将自己的应用打包成镜像，从而让应用在容器中运行。还可以对官方镜像进行扩展，打包成适合生产环境的应用镜像。

---

## Dockerfile常用指令

### FROM：指定基础镜像

指定构建新镜像时使用的基础镜像，通常**必须是Dockerfile的第一个有效指令**

```yaml
# 格式   
FROM <image:[版本标签]>

# 指定基础镜像
FROM centos
FROM ubuntu:20.04  # 说明：**如果不指定版本标签，默认使用latest**
```

### LABEl：添加镜像的元数据

LABEL一般使用键值对方式。

```yaml
# 格式
LABEL <key>=<value> <key>=<value>....

# 为镜像添加元数据
LABEL version="2.0" description="业务系统第二个版本"  # 说明：一条LABEL可以指定多条元数据，尽量不要写多个LABEL
```

### RUN：构建镜像时执行的命令

有两种执行方式

**方式一：shell执行**

```yaml
# 格式
RUN <command>

# 示例
RUN yum update -y  

# 说明：RUN 指令创建的中间镜像会被缓存，并在下次构建中使用。可以在构建时使用--no-cache取消缓存：   docker build --no-cache  
```

**方式二：exec执行**

```yaml
# 格式：
RUN ["executable", "参数1", "参数2"]

# 示例
RUN ["/bin/bash", "-c", "echo hello"]
```

### ENV：在容器内部设置环境变量

```yaml
# 格式
ENV <key> <value>
# 示例
ENV MYPATH /usr/local
```

### ADD：将本地文件添加到镜像中

相当于scp。只是scp 需要将用户名和密码的权限验证，而ADD 不用。

tar 类型文件会自动解压，可以访问网络资源，类似 wget

```yaml
# 语法
ADD <src> <dest>
ADD ["src", "dest"]

# 示例
COPY nginx.tar /opt  

# 说明：路径可以是相对路径或绝对路径，建议使用绝对路径。   
# 尽量不要把<src> 写成一个文件夹，如果<src> 是一个文件夹，复制整个目录的内容
```

### COPY：将文件或目录复制到镜像中。

**功能类似 ADD，但不会自动解压文件，也不能访问网络资源**

```yaml
# 语法
COPY <src> <dest>
COPY ["<src>", "<dest>"]
```

与 ADD 区别，**COPY 的****只能是本地文件**，其他用法一致

### VOLUME：指定持久化目录

```yaml
# 格式
VOLUME ["/path/to/dir"]

# 示例
VOLUME ["/data"]
VOLUME ["/data/log", "/data/config"]   
# 一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统
```

容器使用的是AUFS（联合文件系统），这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失。所以**如果想要数据持久化，就得使用VOLUME**

**功能说明：**卷可以在容器间共享和重用；修改卷后会立即生效；对卷的修改不会对镜像产生影响

### CMD：容器启动时运行的命令（不常用）

```yaml
# 语法
CMD ["executable","param1","param2"]
CMD command param1 param2
# 示例
CMD echo $LANG
```

### ENTRYPOINT：容器启动时运行的命令（常用）

```yaml
# 语法
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2

# 示例
ENTRYPOINT java -jar /data/config.jar
```

ENTRYPOINT和CMD的异同点：

**相同点：**

（1）只能写一条，如果写了多条，那么只有最后一条生效

（2）容器启动时才运行，运行时机相同

**不同点：**

（1）**ENTRYPOINT不会被运行的command覆盖，而CMD则会被覆盖**

（2）如果我们在Dockerfile种同时写了ENTRYPOINT和CMD，并且CMD指令不是一个完整的可执行命令，那么CMD指定的内容将会作为ENTRYPOINT的参数

### EXPOSE：暴露容器运行时的监听端口给外部

```yaml
# 格式：
EXPOSE <port> [<port>...]

# 示例：
EXPOSE 8080
EXPOSE 443/tcp 80/tcp
```

**注意：**

**EXPOSE 并不会让容器的端口访问到主机。**在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口

如果没有暴露端口，后期也可以通过**docker run -p 8080:8080 方式映射端口**，但不能通过 -P 形式映射

### WORKDIR：工作目录

功能类似cd命令。后续指令将在该目录中执行。

### 其他指令

MAINTAINER：指定作者

USER：设置启动容器的用户

ONBUILD：用于设置镜像触发器

HEALTHCHECK：用于检查容器健康状况


## 制作镜像实例（重点）

### 制作镜像语法
----------

`docker build -t 镜像名:标签名 Dockerfile路径（可以是绝对路径，也可以是相对路径）`

### 实例1：构建nginx镜像

**（1）编写Dockerfile**

```bash
[root@harbor ~]# cd /data/dockerfile/nginx/    
[root@harbor nginx]# cat Dockerfile    
# 使用官方Nginx镜像
FROM nginx:1.25
# 复制自定义配置文件
COPY nginx.conf /etc/nginx/nginx.conf
# 复制网站文件
COPY html /usr/share/nginx/html
# 暴露Nginx端口
EXPOSE 80
```

**（2）编写自动配置文件和网站文件**

这里根据你自己情况来编写，放在dockerfile同级目录下

```bash
root@kelvyn:~/nginx# tree
.
├── Dockerfile
├── html
└── nginx.conf

```

**（3）构建镜像**

如果是使用官方镜像的话，会出现以下错误，拉不到镜像

**最近dockerhub不稳定**，比较麻烦!

**解决镜像问题**：

使用公开的镜像加速服务：**docker.m.daocloud.io/**

**第一步：拉取nginx镜像**

需要跟dockerfile中同版本

`docker pull docker.m.daocloud.io/nginx:1.25` 

**第二步：修改标签**

`docker tag docker.m.daocloud.io/nginx:1.25  nginx:1.25` 

完成以上两步就可以再次执行build过程了


（4）运行容器

`docker run -d -p 80:80 --name mynginx mynginx   `

### 实例2：构建 Java 应用镜像

**（1）准备工作**

确保你的Java应用已经打包成JAR文件，并放在Dockerfile同级目录下

**（2）编写Dockerfile**

```yaml
FROM openjdk:8
MAINTAINER liyb <liyb@gdhmail.com>
RUN cp -a-f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \    
    mkdir /work 
COPY *.jar /work/app.jar 
EXPOSE 9999
ENTRYPOINT java -jar /work/app.jar
```

**（3）构建镜像**

如遇无法下载基础镜像的问题，跟上面实例做法一样。

`docker bulid -t myapp . `

**（4）运行容器**

`docker run -d -p 8080:8080 myapp`

## Dockerfile优化技巧

在编写高效的Dockerfile时，可以遵循以下最佳实践，以便优化镜像的构建和运行性能：

### 使用轻量基础镜像

选择小巧的基础镜像，如alpine，以减少最终镜像的体积。

### 合并命令

每个指令（如 RUN、COPY 等）都会生成一个新的镜像层。使用&&将多个RUN命令合并，减少镜像层数，从而提高构建速度和效率。

如下面例子

```yaml
RUN wget http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz \    
    && tar -zxvf nginx-$NGINX_VERSION.tar.gz \
    && cd nginx-$NGINX_VERSION \
    && ./configure --prefix=/usr/local/nginx --with-http_ssl_module \
    && make \
    && make install \
```

### 缓存优化：

合理地安排指令顺序。将不常变化的指令放在Dockerfile前面，常变化的放在后面，以利用缓存，加快构建速度。

### 使用.dockerignore：

排除不必要的文件，减小上下文大小，提升构建速度。

### 多阶段构建

利用多阶段构建，仅将最终运行所需的文件复制到最终镜像，降低镜像大小。

### 优化入口点和命令

使用ENTRYPOINT和CMD正确配置容器启动方式，确保灵活性和一致性。

### 按需复制文件

只复制构建所需的文件，而不是整个项目

以上是我对dockerfile的一些知识的总结和理解，**如有错漏，敬请指出！**

> 转载自： 运维李哥不背锅 [https://mp.weixin.qq.com/s/Mow5BxG93Bz9WuXe85sHNw](https://mp.weixin.qq.com/s/Mow5BxG93Bz9WuXe85sHNw)，如有侵权，请联系删除。