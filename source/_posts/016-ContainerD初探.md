---
title: containerd初探
categories:
  - 笔记
tags:
  - Docker
  - Containerd
abbrlink: 100016
date: 2024-12-6 20:43:17
updated: 
sticky: 
---

# 什么是Containerd？

我这里就简单的讲一点，不过多的赘述。

当时Docker的容器化技术崛起，在这条赛道上无敌，Google在容器编排上搞出了Kubernetes，Docker自己搞出了Swarm，但是k8s一家独大。Docker把自己的Containerd捐给了CNCF(Cloud Native Computing Fundation)。

Kubernetes为了表示中立性，搞一个标准化的容器运行时接口CRI(Container Runntime Interface)，Containerd是第一个支持的。经过这几年这么多版本的迭代，Containerd越来越健壮，它的口号是simplicity, robustness and portability（简单、健壮、可移植）。

总而言之，docker被自己和其他大佬基本给玩死了。现在Containerd是完全可以替代docker这项技术的，而且在k8s中可以原生支持，所以我们没有不学习的理由，况且，这玩意的用法有了nerdctl后，跟docker的用法基本一模一样。

# 安装Containerd

首先，我们应该知道官网：[https://containerd.io/](https://containerd.io/)

我们会用到的几个软件的项目地址：
- Containerd: [https://github.com/containerd/containerd](https://github.com/containerd/containerd)
- runc: [https://github.com/containerd/containerd](https://github.com/containerd/containerd)
- CNI plugins: [https://github.com/containernetworking/plugins](https://github.com/containernetworking/plugins)
- nerdctl: [https://github.com/containerd/nerdctl](https://github.com/containerd/nerdctl)
- buildkit: [https://github.com/moby/buildkit](https://github.com/moby/buildkit)
  - 这个东西因为`nerdctl build`依赖它，主要是用于从dockerfile来build image的，建议安装

## 第一种方法

第一个方法是去每个项目中单独下载相应的包，然后安装进系统中。

 - 安装Containerd

```bash
wget https://github.com/containerd/containerd/releases/download/v2.0.0/containerd-2.0.0-linux-amd64.tar.gz

tar zxvf containerd-2.0.0-linux-amd64.tar.gz -C /usr/local

tee /usr/local/lib/systemd/system/containerd.service << EOF
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target dbus.service

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target

EOF


systemctl daemon-reload
systemctl enable containerd --now
```

- 安装runc

```bash
wget https://github.com/opencontainers/runc/releases/download/v1.2.2/runc.amd64

install -m 755 runc.amd64 /usr/local/sbin/runc
```

- 安装CNI plugins

```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.6.1/cni-plugins-linux-amd64-v1.6.1.tgz
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.6.1.tgz
```

- cli工具

| Name    | Community             | API    | Target             | Web site                                                                |
|---------|-----------------------|--------|--------------------|-------------------------------------------------------------------------|
| ctr     | containerd            | Native | For debugging only | (None, see ctr --help to learn the usage)                               |
| nerdctl | containerd (non-core) | Native | General-purpose    | https://github.com/containerd/nerdctl                                   |
| crictl  | Kubernetes SIG-node   | CRI    | For debugging only | https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md |
|         |                       |        |                    |                                                                         |

这里我推荐使用更加友好的`nerdctl`。

> ps: nerdctl `2.0.0及2.0.2`版本对于`docker.io`的域名解析有问题，导致无法使用镜像加速，对国内用户来说影响太大，所以不推荐使用这两个版本。官方回复，他们将在`2.0.2`版本解决这个问题。

nerdctl命令自动补全

```bash
echo 'source <(nerdctl completion bash)' >> /etc/profile
source /etc/profile

# 查看版本
nerdctl version
```

- 还需要安装`buildkit`

`buildkitd` 有两种可用的 `worker`，一个是 `runc` ，一个是 `containerd` ；默认使用 `runc` ，在 `buildkitd` 参数中为 `oci-worker`。

使用 `containerd` 作为 `worker`，需要增加 `--oci-worker=false --containerd-worker=true` 参数。

```bash
wget https://github.com/moby/buildkit/releases/download/v0.18.1/buildkit-v0.18.1.linux-amd64.tar.gz
tar Czxvf /usr/local buildkit-v0.18.1.linux-amd64.tar.gz

# tee /usr/local/lib/systemd/system/buildkit.socket <<EOF
# [Unit]
# Description=BuildKit
# Documentation=https://github.com/moby/buildkit

# [Socket]
# ListenStream=%t/buildkit/buildkitd.sock
# SocketMode=0660

# [Install]
# WantedBy=sockets.target
# EOF

tee /usr/local/lib/systemd/system/buildkit.service <<EOF
[Unit]
Description=BuildKit
# Requires=buildkit.socket
# After=buildkit.socket
Documentation=https://github.com/moby/buildkit

[Service]
Type=notify
ExecStart=/usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true

[Install]
WantedBy=multi-user.target
EOF


systemctl daemon-reload
systemctl enabled buildkit --now
```

## 第二种方法（推荐）

第二种就会很简单，下载nerdctl-full的包。

```bash
wget https://github.com/containerd/nerdctl/releases/download/v1.7.7/nerdctl-full-1.7.7-linux-amd64.tar.gz
tar Cxzvvf /usr/local nerdctl-full-1.7.7-linux-amd64.tar.gz

systemctl daemon-reload
systemctl enabled buildkit --now
```

如果在root模式使用：

```bash
$ sudo systemctl enable --now containerd
$ sudo nerdctl run -d --name nginx -p 80:80 nginx:alpine
```

如果在非root模式（rootless）下使用：

```bash
$ containerd-rootless-setuptool.sh install
$ nerdctl run -d --name nginx -p 8080:80 nginx:alpine
```

# 镜像加速

> 目前很多文章抄来抄去的，把很多本来很好的文章全整烂了，无奈。
> 就比如这个Containerd加速，k8s和单纯containerd的加速配置不是完全一样的。下边我来讲。

## containerd

如果你使用`nerdctl`，那么你可以直接配置`/etc/containerd/certs.d`，因为`nerdctl`和`ctr`没有用到`cri`，所以不需要配置文件`/etc/containerd/config.toml`。

`nerdctl`和`ctr`有一个参数`--hosts-dir`，这个指引着cli tool通过镜像加速下载image。

所以我们配置好`/etc/containerd/certs.d`就可以了。

```bash
mdkir -p /etc/containerd/certs.d/docker.io

cat > /etc/containerd/certs.d/docker.io/hosts.toml << EOF
server = "https://docker.io"

[host."https://docker.wanpeng.top"]
  capabilities = ["pull", "resolve"]

[host."docker.unsee.tech"]
  capabilities = ["pull", "resolve"]

[host."https://docker.m.daocloud.io"]
  capabilities = ["pull", "resolve"]

EOF
```

验证（使用参数 `--debug=true` ，查看是否使用了镜像加速地址）：

```bash
# ctr 需要指定 --hosts-dir
ctr --debug=true images pull docker.io/library/nginx:latest --hosts-dir=/etc/containerd/certs.d


# nerdctl 不需要指定，因为它 hosts_dir 默认的值是  [/etc/containerd/certs.d,/etc/docker/certs.d]
nerdctl --debug=true image pull docker.io/library/nginx:latest

* * *

# ctr --debug=true images pull docker.io/library/nginx:latest --hosts-dir=/etc/containerd/certs.d
DEBU[0000] fetching                  image="docker.io/library/nginx:latest"
DEBU[0000] loading host directory    dir=/etc/containerd/certs.d/docker.io
DEBU[0000] resolving                 host=docker.wanpeng.top
DEBU[0000] do request                host=docker.wanpeng.top request.header.accept=（...后边不写了）
```

## k8s

k8s使用 `crictl` 所以需要在 Containerd 的配置文件中指定 `config_path`

```bash
containerd config default > /etc/containerd/config.toml

vim /etc/containerd/config.toml

# 改成下边的
[plugins."io.containerd.grpc.v1.cri".registry]
      config_path = "/etc/containerd/certs.d"

systemctl restart containerd
```

## `/etc/containerd/certs.d` 怎么写

不保证下面的镜像加速地址能用~~~

```bash
# docker.io 镜像加速
mdkir -p /etc/containerd/certs.d/docker.io

tee /etc/containerd/certs.d/docker.io/hosts.toml << 'EOF'
server = "https://docker.io"

[host."https://docker.wanpeng.top"]
  capabilities = ["pull", "resolve"]

[host."docker.unsee.tech"]
  capabilities = ["pull", "resolve"]

[host."https://docker.m.daocloud.io"]
  capabilities = ["pull", "resolve"]

EOF

# registry.k8s.io镜像加速
mkdir -p /etc/containerd/certs.d/registry.k8s.io
tee /etc/containerd/certs.d/registry.k8s.io/hosts.toml << 'EOF'
server = "https://registry.k8s.io"

[host."https://k8s.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# docker.elastic.co镜像加速
mkdir -p /etc/containerd/certs.d/docker.elastic.co
tee /etc/containerd/certs.d/docker.elastic.co/hosts.toml << 'EOF'
server = "https://docker.elastic.co"

[host."https://elastic.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# gcr.io镜像加速
mkdir -p /etc/containerd/certs.d/gcr.io
tee /etc/containerd/certs.d/gcr.io/hosts.toml << 'EOF'
server = "https://gcr.io"

[host."https://gcr.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# ghcr.io镜像加速
mkdir -p /etc/containerd/certs.d/ghcr.io
tee /etc/containerd/certs.d/ghcr.io/hosts.toml << 'EOF'
server = "https://ghcr.io"

[host."https://ghcr.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# k8s.gcr.io镜像加速
mkdir -p /etc/containerd/certs.d/k8s.gcr.io
tee /etc/containerd/certs.d/k8s.gcr.io/hosts.toml << 'EOF'
server = "https://k8s.gcr.io"

[host."https://k8s-gcr.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# mcr.m.daocloud.io镜像加速
mkdir -p /etc/containerd/certs.d/mcr.microsoft.com
tee /etc/containerd/certs.d/mcr.microsoft.com/hosts.toml << 'EOF'
server = "https://mcr.microsoft.com"

[host."https://mcr.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# nvcr.io镜像加速
mkdir -p /etc/containerd/certs.d/nvcr.io
tee /etc/containerd/certs.d/nvcr.io/hosts.toml << 'EOF'
server = "https://nvcr.io"

[host."https://nvcr.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# quay.io镜像加速
mkdir -p /etc/containerd/certs.d/quay.io
tee /etc/containerd/certs.d/quay.io/hosts.toml << 'EOF'
server = "https://quay.io"

[host."https://quay.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# registry.jujucharms.com镜像加速
mkdir -p /etc/containerd/certs.d/registry.jujucharms.com
tee /etc/containerd/certs.d/registry.jujucharms.com/hosts.toml << 'EOF'
server = "https://registry.jujucharms.com"

[host."https://jujucharms.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

# rocks.canonical.com镜像加速
mkdir -p /etc/containerd/certs.d/rocks.canonical.com
tee /etc/containerd/certs.d/rocks.canonical.com/hosts.toml << 'EOF'
server = "https://rocks.canonical.com"

[host."https://rocks-canonical.m.daocloud.io"]
  capabilities = ["pull", "resolve", "push"]
EOF
```