---
title: 在RockyLinux9上基于K3S安装AWX
categories:
  - 笔记
tags:
  - Linux
  - AWX
  - Ansible
abbrlink: 17933
date: 2023-02-25 10:51:46
---

{% note success simple %}
快速部署AWX至k8s上，这是一个非常基础的部署，请根据您的具体需求去查看相应的官方文档。
{% endnote %}

## 基础环境
- RockyLinux 9.1
- k3s
部署时有一个google镜像可能需要您自行下载并导入。

## 安装Kubernetes

快速安装单节点的k3s。

```bash
curl -sfL https://get.k3s.io | sh -

# 国内用户请使用以下方法加速安装：
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

验证安装

```bash
$ kubectl get pods -A

NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   local-path-provisioner-79f67d76f8-7qnmp   1/1     Running     0          78s
kube-system   coredns-597584b69b-t5rzs                  1/1     Running     0          78s
kube-system   helm-install-traefik-crd-lxgwd            0/1     Completed   0          78s
kube-system   helm-install-traefik-h98hh                0/1     Completed   1          78s
kube-system   metrics-server-5f9f776df5-wjftg           1/1     Running     0          78s
kube-system   svclb-traefik-a4e4ab03-bz7mb              2/2     Running     0          37s
kube-system   traefik-66c46d954f-rs8jl                  1/1     Running     0          37s
```

## 安装AWX

{% note success simple %}
AWX-oprator官方Github仓库地址：https://github.com/ansible/awx-operator
{% endnote %}

安装[Git](https://git-scm.com/)（必需）

```bash
dnf install git -y
```

安装[kustomize](https://kubectl.docs.kubernetes.io/zh/installation/binaries/)

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash

也可以直接去Github项目中去下载相应的二进制文件：
https://github.com/kubernetes-sigs/kustomize/releases
```
将kustomize的二进制文件移动至`/usr/local/bin/`中，并赋予执行权限。

```bash
mv kustomize /usr/local/bin/

chmod +x /usr/local/bin/kustomize
```

创建kustomization.yaml文件：`vim kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=1.1.4 # 更改你想安装版本号。截止写文章的时候最新版本是1.2.0，但是安装报错，改为1.1.4就没问题了
 
# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 1.1.4 # 与上方保持一致即可

# Specify a custom namespace in which to install AWX
namespace: awx
```
运行以下命令进行初始部署
```bash
$ kustomize build . | kubectl apply -f -
namespace/awx created
customresourcedefinition.apiextensions.k8s.io/awxbackups.awx.ansible.com created
customresourcedefinition.apiextensions.k8s.io/awxrestores.awx.ansible.com created
customresourcedefinition.apiextensions.k8s.io/awxs.awx.ansible.com created
serviceaccount/awx-operator-controller-manager created
role.rbac.authorization.k8s.io/awx-operator-awx-manager-role created
role.rbac.authorization.k8s.io/awx-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/awx-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/awx-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/awx-operator-awx-manager-rolebinding created
rolebinding.rbac.authorization.k8s.io/awx-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/awx-operator-proxy-rolebinding created
configmap/awx-operator-awx-manager-config created
service/awx-operator-controller-manager-metrics-service created
deployment.apps/awx-operator-controller-manager created
```

验证部署

```bash
# 后续所使用的命名空间都是awx
# 这里直接部署会拉取image失败，请自行解决网络问题，或者从别的机器上导出相应的image并导入，image名称：
# gcr.io/kubebuilder/kube-rbac-proxy:v0.13.0

$ kubectl get pods -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-66ccd8f997-rhd4z   2/2     Running   0          11s
```

创建第二个yaml文件，即awx.yaml

```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
```

在之前创建的kustomization.yaml文件中resources里添加一行

```yaml
...
resources:
  - github.com/ansible/awx-operator/config/default?ref=1.1.4
  # 添加以下一行:
  - awx.yaml
...
```
再次运行kustomize创建AWX instance。这个过程需要持续几分钟，快慢取决于你所使用的机器资源配置
```bash
kustomize build . | kubectl apply -f -
```

可以通过命令查看部署进度

```bash
kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager -n awx
```

创建完成后，最终将看到以下信息：

```shell

----- Ansible Task Status Event StdOut (awx.ansible.com/v1beta1, Kind=AWX, awx/awx) -----


PLAY RECAP *********************************************************************
localhost                  : ok=75   changed=0    unreachable=0    failed=0    skipped=71   rescued=0    ignored=1


----------
```

获取登录密码

```bash
$  kubectl get secret awx-admin-password -o jsonpath="{.data.password}" -n awx | base64 --decode ; echo
t6yREMgWH9mJuegVXndRDKWdELaVcWow
```

打开浏览器就可以访问你的AWX了。
- Web地址：http://Your_AWX_IP:30080/#/login
- 用户名：admin
- 密码：上边命令获取的密码