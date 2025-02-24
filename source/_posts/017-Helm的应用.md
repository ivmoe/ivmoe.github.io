---
title: Helm 的应用
categories:
  - 笔记
tags:
  - Kubernetes
  - Helm
abbrlink: 100017
date: 2025-2-24 16:22:06
updated: 2025-2-24 16:22:12
sticky: 
---

# Helm简介
:::info
官网：[https://helm.sh/zh/](https://helm.sh/zh/)

仓库：[https://github.com/helm/helm](https://github.com/helm/helm)

:::

Helm 是 Kubernetes 的包管理器，类似CentOS 的yum、dnf，类似Ubuntu的apt。概括起来就是**优雅**。



Helm 工作流程的概述：

> helm有以下几个核心功能:
>
>         (1)将变量从"value.yaml"文件中获取，并渲染到chart模板文件中;
>
>         (2)chart模板文件对应的是一系列yaml文件，会基于这些yaml清单来部署应用到kuberntes集群;
>
>         (3)helm也有其对应的Chart仓库，这些公共仓库是一些其他开发或者运维人员编写好的chart，如果他们写的chart在我们工作中能用到就是最好的了;
>

# Helm安装
## 下载及安装
**方式一：**使用脚本安装

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

**方式二：下载二进制文件安装**

```bash
# 下载 需要的版本
wget -O helm-v3.0.0-linux-amd64.tar.gz https://get.helm.sh/helm-v3.17.1-linux-amd64.tar.gz

# 解压
tar -zxvf helm-v3.0.0-linux-amd64.tar.gz

# 在解压目录中找到helm程序，移动到需要的目录中
mv linux-amd64/helm /usr/local/bin/helm
```

## helm命令自动补全
```bash
$ helm completion bash > /etc/bash_completion.d/helm && source /etc/bash_completion.d/helm
```

# 使用Helm创建Chart
## Chart 目录结构
创建一个Chart：

```bash
# helm create [Chart's Name]
$ helm create kelvyn-chart
```

目录结构：

```bash
$ tree kelvyn-chart/

kelvyn-chart/
├── charts
├── Chart.yaml    # 包含了chart信息的YAML文件
├── templates			# 模板目录，当和values 结合时，生成有效的Kubernetes文件
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt		# 可选: 包含简要使用说明的纯文本文件
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml			# chart 默认的配置值

---

官方文档的说明：
wordpress/
  Chart.yaml          # 包含了chart信息的YAML文件
  LICENSE             # 可选: 包含chart许可证的纯文本文件
  README.md           # 可选: 可读的README文件
  values.yaml         # chart 默认的配置值
  values.schema.json  # 可选: 一个使用JSON结构的values.yaml文件
  charts/             # 包含chart依赖的其他chart
  crds/               # 自定义资源的定义
  templates/          # 模板目录， 当和values 结合时，可生成有效的Kubernetes manifest文件
  templates/NOTES.txt # 可选: 包含简要使用说明的纯文本文件
```

Helm保留使用 charts/，crds/， templates/目录，以及列举出的文件名。其他文件保持原样。

## Chart.yaml 文件说明
`Chart.yaml</font>` 文件是chart必需的。包含了以下字段：

```yaml
apiVersion: chart API 版本 （必需）
name: chart名称 （必需）
version: 语义化2 版本（必需）
kubeVersion: 兼容Kubernetes版本的语义化版本（可选）
description: 一句话对这个项目的描述（可选）
type: chart类型 （可选）
keywords:
  - 关于项目的一组关键字（可选）
home: 项目home页面的URL （可选）
sources:
  - 项目源码的URL列表（可选）
dependencies: # chart 必要条件列表 （可选）
  - name: chart名称 (nginx)
    version: chart版本 ("1.2.3")
    repository: （可选）仓库URL ("https://example.com/charts") 或别名 ("@repo-name")
    condition: （可选） 解析为布尔值的yaml路径，用于启用/禁用chart (e.g. subchart1.enabled )
    tags: # （可选）
      - 用于一次启用/禁用 一组chart的tag
    import-values: # （可选）
      - ImportValue 保存源值到导入父键的映射。每项可以是字符串或者一对子/父列表项
    alias: （可选） chart中使用的别名。当你要多次添加相同的chart时会很有用
maintainers: # （可选）
  - name: 维护者名字 （每个维护者都需要）
    email: 维护者邮箱 （每个维护者可选）
    url: 维护者URL （每个维护者可选）
icon: 用做icon的SVG或PNG图片URL （可选）
appVersion: 包含的应用版本（可选）。不需要是语义化，建议使用引号
deprecated: 不被推荐的chart （可选，布尔值）
annotations:
  example: 按名称输入的批注列表 （可选）.
```

从 v3.3.2，不再允许额外的字段。推荐的方法是在 annotations 中添加自定义元数据。

## 模板文件说明
模板文件遵守书写Go模板的标准惯例（查看 [文本/模板 Go 包文档](https://golang.org/pkg/text/template/)了解更多）。 模板文件的例子看起来像这样：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: deis-database
  namespace: deis
  labels:
    app.kubernetes.io/managed-by: deis
spec:
  replicas: 1
  selector:
    app.kubernetes.io/name: deis-database
  template:
    metadata:
      labels:
        app.kubernetes.io/name: deis-database
    spec:
      serviceAccount: deis-database
      containers:
        - name: deis-database
          image: {{ .Values.imageRegistry }}/postgres:{{ .Values.dockerTag }}
          imagePullPolicy: {{ .Values.pullPolicy }}
          ports:
            - containerPort: 5432
          env:
            - name: DATABASE_STORAGE
              value: {{ default "minio" .Values.storage }}
```

上面的例子，松散地基于 [https://github.com/deis/charts](https://github.com/deis/charts)， 是一个Kubernetes副本控制器的模板。可以使用下面四种模板值（一般被定义在values.yaml文件）：



imageRegistry: Docker镜像的源注册表

dockerTag: Docker镜像的tag

pullPolicy: Kubernetes的拉取策略

storage: 后台存储，默认设置为"minio"

所有的值都是模板作者定义的。Helm不需要或指定参数。

## 预定义的values
alues通过模板中.Values对象可访问的values.yaml文件（或者通过 --set 参数)提供， 但可以模板中访问其他预定义的数据片段。



以下值是预定义的，对每个模板都有效，并且可以被覆盖。和所有值一样，名称 区分大小写。



+ `Release.Name`: 版本名称(非chart的)
+ `Release.Namespace`: 发布的chart版本的命名空间
+ `Release.Service`: 组织版本的服务
+ `Release.IsUpgrade`: 如果当前操作是升级或回滚，设置为true
+ `Release.IsInstall`: 如果当前操作是安装，设置为true
+ `Chart`: `Chart.yaml`的内容。因此，chart的版本可以从 Chart.Version 获得， 并且维护者在Chart.Maintainers里。
+ `Files`: chart中的包含了非特殊文件的类图对象。这将不允许您访问模板， 但是可以访问现有的其他文件（除非被.helmignore排除在外）。 使用{{ index .Files "file.name" }}可以访问文件或者使用{{.Files.Get name }}功能。 您也可以使用{{ .Files.GetBytes }}作为[]byte访问文件内容。
+ `Capabilities`: 包含了Kubernetes版本信息的类图对象。({{ .Capabilities.KubeVersion }}) 和支持的Kubernetes API 版本({{ .Capabilities.APIVersions.Has "batch/v1" }})

**注意：** 任何未知的Chart.yaml字段会被抛弃。在Chart对象中无法访问。因此， Chart.yaml不能用于将任意结构的数据传递到模板中。不过values文件可用于此。

## values.yaml 文件说明
考虑到前面部分的模板，`values.yam`文件提供的必要值如下：

```yaml
imageRegistry: "quay.io/deis"
dockerTag: "latest"
pullPolicy: "Always"
storage: "s3"
```

# 基于 Helm 部署应用
## 创建文件
首先初始化以下文件结构：

```bash
$ mkdir -p /helm_data && cd /helm_data 
$ helm create kelvyn-chart
$ cd kelvyn-chart/templates/
rm -rf *.yaml tests

```

Chart.yaml

```yaml
apiVersion: v2
name: kelvyn-chart
description: Kelvyn Nginx Demo Chart.
type: application
version: 0.1.0
appVersion: "1.16.0"

```

values.yaml

```yaml
podLabel:
  key: app
  value: nginx-deploy
replicas: 2
images:
  image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx
  imageTag: 1.27.3-alpine
  imagePullPolicy: IfNotPresent

service:
  type: NodePort
  nodePort: 30060
```

templates/nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: kelvyn-helm

spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      {{ .Values.podLabel.key }}: {{ .Values.podLabel.value }}
  template:
    metadata:
      name: nginx
      labels:
        {{ .Values.podLabel.key }}: {{ .Values.podLabel.value }}
    spec:
      containers:
        - name: nginx
          image: {{ .Values.images.image }}:{{ .Values.images.imageTag }}
          imagePullPolicy: {{ .Values.images.imagePullPolicy }}
          args: ["sh", "-c", "echo 111 >> /usr/share/nginx/html/index.html && nginx -g 'daemon off;'"]
          ports:
            - containerPort: 80
              protocol: TCP
          resources:
            limits:
              cpu: 100m
              memory: 100Mi
            requests:
              cpu: 100m
              memory: 100Mi
```

templates/nginx-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-deploysvc
  namespace: kelvyn-helm
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: 80
      targetPort: 80
      nodePort: {{ .Values.service.nodePort }}
  selector:
    {{ .Values.podLabel.key }}: {{ .Values.podLabel.value }}
```

templates/NOTES.txt

```plain
This is Kelvyn's Test Helm Package, Welcome to Test!!!
```

最后目录结构：

```bash
$ tree
.
├── charts
├── Chart.yaml
├── templates
│   ├── _helpers.tpl
│   ├── nginx-deployment.yaml
│   ├── nginx-service.yaml
│   └── NOTES.txt
└── values.yaml

2 directories, 6 files

```

## 使用 Helm Cli 部署应用
```bash
# 创建namespace
$ kubectl create ns kelvyn-helm

# 部署应用
$ helm install kelvyn-nginx /helm_data/kelvyn-chart/ -n kelvyn-helm
    
    # NAME: kelvyn-nginx
    # LAST DEPLOYED: Sun Feb 23 20:44:21 2025
    # NAMESPACE: kelvyn-helm
    # STATUS: deployed
    # REVISION: 1
    # TEST SUITE: None
    # NOTES:
    # This is Kelvyn's Test Helm Package, Welcome to Test!!!

# 查看 Helm 部署的应用
$ helm list -n kelvyn-helm

    # NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
    # kelvyn-nginx    kelvyn-helm     1               2025-02-23 20:44:21.241661693 +0800 CST deployed        kelvyn-chart-0.1.0      1.16.0

$ kubectl -n kelvyn-helm get all

# 卸载应用
$ helm uninstall kelvyn-nginx -n kelvyn-helm
```

在 `helm install`阶段，可以使用参数 `--dry-run`进行测试安装

使用了"--dry-run"选项并未真正部署!它会列出要执行的yaml文件内容!

```bash
$ helm install kelvyn-nginx /helm_data/kelvyn-chart/ -n kelvyn-helm --dry-run

NAME: kelvyn-nginx
LAST DEPLOYED: Sun Feb 23 20:51:01 2025
NAMESPACE: kelvyn-helm
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: kelvyn-chart/templates/nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-deploysvc
  namespace: kelvyn-helm
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30060
  selector:
    app: nginx-deploy
---
# Source: kelvyn-chart/templates/nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: kelvyn-helm

spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-deploy
  template:
    metadata:
      name: nginx
      labels:
        app: nginx-deploy
    spec:
      containers:
        - name: nginx
          image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:1.27.3-alpine
          imagePullPolicy: IfNotPresent
          args: ["sh", "-c", "echo 111 >> /usr/share/nginx/html/index.html && nginx -g 'daemon off;'"]
          ports:
            - containerPort: 80
              protocol: TCP
          resources:
            limits:
              cpu: 100m
              memory: 100Mi
            requests:
              cpu: 100m
              memory: 100Mi

NOTES:
This is Kelvyn's Test Helm Package, Welcome to Test!!!
```

# 基于 Helm 升级 Chart 的 Release
:::info
**官方文档：** [https://helm.sh/zh/docs/helm/helm_upgrade/](https://helm.sh/zh/docs/helm/helm_upgrade/)

:::

## 概述
```latex
为了实现Chart复用，可动态传参修改"values.yaml"文件中的变量值，有以下两种方式:
	--values,-f:
		基于yaml配置文件方式升级Release。
		例如:"helm upgrade  -f /oldboyedu/data/oldboyedu-nginx/values.yaml myweb02 /oldboyedu/data/oldboyedu-nginx"
		
	--set:
		基于命令行方式升级Release。例如:"helm upgrade --set imageTag=1.18 myweb02 /oldboyedu/data/oldboyedu-nginx"

    如下图所示，对比了基于yaml配置文件和基于命令行方式升级Release的差别。
```

![](https://cdn.nlark.com/yuque/0/2025/png/2921054/1740316382671-a0adc20d-b7b7-4429-a014-78f77f819222.png)

## 基于配置文件升级
`<font style="color:rgb(0, 0, 0);background-color:rgb(240, 243, 243);">helm upgrade</font>`使用 `-f`参数升级：



可以多次指定'--values'/'-f'参数，最后（最右边）指定的文件优先级最高。比如如果myvalues.yaml和override.yaml同时包含了名为 'Test'的key，override.yaml中的设置会优先使用：

`$ helm upgrade -f myvalues.yaml -f override.yaml redis ./redis`

实战：

```bash
# 查看应用的历史版本记录
$ helm history kelvyn-nginx -n kelvyn-helm
  REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
  1               Sun Feb 23 21:00:50 2025        deployed        kelvyn-chart-0.1.0      1.16.0          Install complete

#使用curl查看Nginx版本
$ kubectl -n kelvyn-helm get svc nginx-deploysvc
  NAME              TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
  nginx-deploysvc   NodePort   10.98.56.127   <none>        80:30060/TCP   16m

$ curl -I 10.98.56.127
  HTTP/1.1 200 OK
  Server: nginx/1.27.3
  Date: Sun, 23 Feb 2025 13:17:21 GMT
  Content-Type: text/html
  Content-Length: 619
  Last-Modified: Sun, 23 Feb 2025 13:00:51 GMT
  Connection: keep-alive
  ETag: "67bb1c03-26b"
  Accept-Ranges: bytes

# 将 values.yaml 中 image 的版本修改下
$ sed -i 's/1.27.3-alpine/1.27.4-alpine/' /root/helm/kelvyn-chart/values.yaml

# 更新 Chart
$ helm upgrade -f /root/helm/kelvyn-chart/values.yaml kelvyn-nginx /root/helm/kelvyn-chart/ -n kelvyn-helm

# 验证
$ helm history kelvyn-nginx -n kelvyn-helm
  REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
  1               Sun Feb 23 21:37:03 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Install complete
  2               Sun Feb 23 21:38:37 2025        deployed        kelvyn-chart-0.1.0      1.16.0          Upgrade complete

$ curl -I 10.98.56.127
  HTTP/1.1 200 OK
  Server: nginx/1.27.4
  Date: Sun, 23 Feb 2025 13:41:30 GMT
  Content-Type: text/html
  Content-Length: 619
  Last-Modified: Sun, 23 Feb 2025 13:38:40 GMT
  Connection: keep-alive
  ETag: "67bb24e0-26b"
  Accept-Ranges: bytes

# Nginx 版本已经变成 1.27.4 了，升级成功
```

## 基于命令行升级
以多次指定'--set'参数，最后（最右边）指定的优先级最高。比如'bar' 和 'newbar'都设置了一个名为'foo'的可以， 'newbar'的值会优先使用：

    `$ helm upgrade --set foo=bar --set foo=newbar redis ./redis`

实战：

```bash
# 升级image镜像从1.27.3-alpine到1.27.4-alpine，并将副本数设置为1
$ helm upgrade --set images.imageTag=1.27.4-alpine,replicas=1 kelvyn-nginx ./kelvyn-chart/ -n kelvyn-helm

$ curl -I 10.98.56.127
  HTTP/1.1 200 OK
  Server: nginx/1.27.4
  Date: Sun, 23 Feb 2025 13:41:30 GMT
  Content-Type: text/html
  Content-Length: 619
  Last-Modified: Sun, 23 Feb 2025 13:38:40 GMT
  Connection: keep-alive
  ETag: "67bb24e0-26b"
  Accept-Ranges: bytes
  
# Nginx 版本已经变成 1.27.4 了，升级成功

$ kubectl -n kelvyn-helm get pod -o wide
  NAME                            READY   STATUS    RESTARTS   AGE    IP            NODE       NOMINATED NODE   READINESS GATES
  nginx-deploy-689f6b998b-87m5f   1/1     Running   0          2m8s   10.244.2.57   worker02   <none>           <none>

# 副本数变为了1
```

# 基于 Helm 回滚应用版本
## 回滚到上一个版本
```bash
# 查看历史版本
$ helm history kelvyn-nginx -n kelvyn-helm
REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
1               Sun Feb 23 21:53:34 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Install complete
2               Sun Feb 23 21:55:06 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Upgrade complete
3               Sun Feb 23 21:57:11 2025        deployed        kelvyn-chart-0.1.0      1.16.0          Upgrade complete

# 回滚到上一个版本
$ helm rollback kelvyn-nginx -n kelvyn-helm
  Rollback was a success! Happy Helming!

# 再次查看历史版本
$ helm history kelvyn-nginx -n kelvyn-helm
  REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
  1               Sun Feb 23 21:53:34 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Install complete
  2               Sun Feb 23 21:55:06 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Upgrade complete
  3               Sun Feb 23 21:57:11 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Upgrade complete
  4               Sun Feb 23 22:01:35 2025        deployed        kelvyn-chart-0.1.0      1.16.0          Rollback to 2

# 副本数变回了2
```

![](https://cdn.nlark.com/yuque/0/2025/png/2921054/1740319491727-5890de82-33ad-4b2b-95eb-4b8b32401eec.png)

## 回滚到指定版本
```bash
# 查看历史版本
$ helm history kelvyn-nginx -n kelvyn-helm
  REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
  1               Sun Feb 23 21:53:34 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Install complete
  2               Sun Feb 23 21:55:06 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Upgrade complete
  3               Sun Feb 23 21:57:11 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Upgrade complete
  4               Sun Feb 23 22:01:35 2025        deployed        kelvyn-chart-0.1.0      1.16.0          Rollback to 2

$ helm rollback kelvyn-nginx 1 -n kelvyn-helm
  Rollback was a success! Happy Helming!

# 查看历史版本
$ helm history kelvyn-nginx -n kelvyn-helm
REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
1               Sun Feb 23 21:53:34 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Install complete
2               Sun Feb 23 21:55:06 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Upgrade complete
3               Sun Feb 23 21:57:11 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Upgrade complete
4               Sun Feb 23 22:01:35 2025        superseded      kelvyn-chart-0.1.0      1.16.0          Rollback to 2
5               Sun Feb 23 22:06:13 2025        deployed        kelvyn-chart-0.1.0      1.16.0          Rollback to 1
```

![](https://cdn.nlark.com/yuque/0/2025/png/2921054/1740319689447-48f79f76-e9a1-4477-bbb0-18494721fb99.png)

# 公有仓库管理及使用
## 主流仓库
```latex
国内仓库：
  - 微软巨硬：
    https://mirror.azure.cn/kubernetes/charts/
  
  - 阿里云仓库：
    https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

当然还有一些其他搭建的用户较小的仓库，比如：
  - helm-charts.itboon.top:
    https://helm-charts.itboon.top/docs/
  - https://charts.ost.ai/
    https://charts.ost.ai
```

## 仓库管理
```bash
# 1. 列出所有仓库
$ helm repo list

# 2. 添加一个仓库
$ helm repo add azure https://mirror.azure.cn/kubernetes/charts
$ helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# 3. 更新仓库信息
$ helm repo update  			# 更新所有仓库
$ helm repo update aliyun # 更新指定仓库

# 4. 搜索Chart
$ helm search repo mysql	# 搜索所有仓库中的mysql
$ helm search repo aliyun	# 查看aliyun仓库中的所有包

# 5. 删除一个仓库
$ helm repo remove azure
```

## 实战练习
使用 Helm 公有仓库部署一个 MySQL ：

```bash
# 搜索Chart
$ helm search repo mysql

# 拉取Chart
$ helm pull aliyun/mysql --untar
    # --untar 表示自动解压缩

# 查看Chart目录结构
$ tree mysql/
mysql/
├── Chart.yaml
├── README.md
├── templates
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── NOTES.txt
│   ├── pvc.yaml
│   ├── secrets.yaml
│   └── svc.yaml
└── values.yaml

# 修改 Deployment 的apiVersion 值 "extensions/v1beta1" 为 "apps/v1"
$ sed -i "s/extensions\/v1beta1/apps\/v1/" ./mysql/templates/deployment.yaml

# 执行以下下命令
$ sed -i '10a\
  selector:\
    matchLabels:\
      app: {{ template "mysql.fullname" . }}' ./mysql/templates/deployment.yaml

# 需要手动创建个PV
cat >  ./mysql/templates/pv.yaml << EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kelvyn-mysql-helm-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/tmp/kelvyn-mysql"
EOF

# 安装
$ helm install kelvyn-mysql mysql/ -n kelvyn-mysql-helm --create-namespace

# 测试连接MySQL
$ kubectl get pods -o wide  # 查看MySQL的IP地址
$ mysql -h 10.244.1.35 -p`kubectl get secret --namespace default kelvyn-mysql-mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d`
```

:::color4
**还得记得注意，修改文件 values.yaml 中的 image 值**！！！！！！

:::

# 私有仓库搭建及使用
## 使用 docker 部署 ChartMuseum 私有 Chart 仓库
:::info
**ChartMuseum官网：**[https://chartmuseum.com/](https://chartmuseum.com/)

**项目地址：**[https://github.com/helm/chartmuseum](https://github.com/helm/chartmuseum)

:::

```bash
# 创建持久化目录
$ mkdir -p /data/chartmuseum

# 启动容器
$ docker run -d \
  -p 8090:8080 \
  -e DEBUG=1 \
  -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  -v /data/chartmuseum:/charts \
  --restart=always \
  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/ghcr.io/helm/chartmuseum:v0.16.2
  # ghcr.io/helm/chartmuseum:v0.14.0

# 测试部署是否成功
$ curl http://192.168.1.51:8090/api/charts
$ curl http://192.168.1.51:8090/

# 赋予挂载目录权限 777
$ chmod 777 /data/chartmuseum
```

![](https://cdn.nlark.com/yuque/0/2025/png/2921054/1740382402983-23585fca-18b7-4e43-9e59-c085bb60d124.png)



## 下载公有仓库的 Chart 至私有仓库
```bash
cd /data/chartmuseum

# 直接拉取就好
$ helm pull azure/mysql
$ helm pull azure/redis

# 查看结果
curl http://192.168.1.51:8090/api/charts | jq .
```

```json
{
  "mysql": [
    {
      "name": "mysql",
      "home": "https://www.mysql.com/",
      "sources": [
        "https://github.com/kubernetes/charts",
        "https://github.com/docker-library/mysql"
      ],
      "version": "1.6.9",
      "description": "DEPRECATED - Fast, reliable, scalable, and easy to use open-source relational database system.",
      "keywords": [
        "mysql",
        "database",
        "sql"
      ],
      "icon": "https://www.mysql.com/common/logos/logo-mysql-170x115.png",
      "apiVersion": "v1",
      "appVersion": "5.7.30",
      "deprecated": true,
      "urls": [
        "charts/mysql-1.6.9.tgz"
      ],
      "created": "2025-02-24T07:40:46.457698552Z",
      "digest": "35f232f0b4df50e85c6d65cbe7e5291b8ac8fab2c5a6afa8a0b816c7ce594390"
    }
  ]
},
...
```

## 打包自定义的 Chart
```bash
# 打包自定义的 Chart
$ helm package kelvyn-chart/

# 添加私有仓库
$ helm repo add chartmuseum http://192.168.1.51:8090/

```

## 使用插件 cm-push 推送自定义的 Chart
[https://github.com/chartmuseum/helm-push](https://github.com/chartmuseum/helm-push)

```bash
# 安装git
$ dnf install -y git

# 安装插件
$ helm plugin install https://github.com/chartmuseum/helm-push

# 推送
$ helm cm-push kelvyn-chart-0.1.0.tgz chartmuseum
    Pushing kelvyn-chart-0.1.0.tgz to chartmuseum...
    Done.
```



