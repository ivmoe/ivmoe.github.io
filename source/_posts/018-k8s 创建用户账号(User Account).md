---
title: Kubernetes 创建用户账号(User Account)——包含自动创建脚本
categories:
  - 笔记
tags:
  - Kubernetes
abbrlink: 100018
date: 2025-2-25 22:06:09
updated: 2025-2-25 22:48:12
sticky: 
---

# 创建证书

根据 k8s 集群的 CA 创建用户的证书：

```bash
mkdir -p /k8s-user/kelvyn && cd /k8s-user/kelvyn

# 1. 创建私钥
(umask 077;openssl genrsa -out kelvyn.key 2048)

# 2. 创建签名请求文件
openssl req -new -key kelvyn.key -out kelvyn.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=GE/OU=CT/CN=kelvyn"

# 3. 签发证书（期限一年）
openssl  x509 -req -in kelvyn.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out kelvyn.crt -days 365
```

# 创建配置文件

创建配置文件有以下几个步骤：

> kubectl config set-cluster --kubeconfig=/PATH/TO/SOMEFILE      #集群配置
>
> kubectl config set-credentials NAME --kubeconfig=/PATH/TO/SOMEFILE #用户配置
>
> kubectl config set-context    #context配置
>
> kubectl config use-context    #切换context
>

+ `–embed-certs=true`的作用是不在配置文件中显示证书信息。
+ `–kubeconfig=/k8s-user/kelvyn/kelvyn.conf`

用于创建新的配置文件，如果不加此选项,则内容会添加到家目录下.kube/config文件中，可以使用use-context来切换不同的用户管理k8s集群。可以不加，**我建议添加**。

context简单的理解就是用什么用户来管理哪个集群，即用户和集群的结合。

```bash
# 1. 创建集群配置
kubectl config set-cluster k8s --server=https://192.168.1.55:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --kubeconfig=/k8s-user/kelvyn/kelvyn.conf

# 2. 创建用户配置
kubectl config set-credentials kelvyn --client-certificate=kelvyn.crt --client-key=kelvyn.key --embed-certs=true --kubeconfig=/k8s-user/kelvyn/kelvyn.conf

# 3. 创建 Context 配置
kubectl config set-context kelvyn@k8s --cluster=k8s --user=kelvyn --kubeconfig=/k8s-user/kelvyn/kelvyn.conf

# 4. 切换 Context 配置
kubectl config use-context kelvyn@k8s --kubeconfig=/k8s-user/kelvyn/kelvyn.conf

```

最终生成的配置文件（/k8s-user/kelvyn/kelvyn.conf）

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJWEVueGdWaGlFWE13RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBeU1EZ3dNalF5TWpkYUZ3MHpOVEF5TURZd01qUTNNamRhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURPQ1UyWll0T3dkVE9BWkVTRS90MW9MSnhET00vejBvSFdJVlNIZHBaL2VBTklmVlpxUnFCbGlTVFcKZlA2ak04SEJzeWMyeU01RGtTR2tIUm4wM25IQmpGYWV2aU5SVHBDTWl0UWkwVEhwRFNiM3JWaEtsZjZhSHFrbQpzOHJhaHpyYUkwcTJESTVvQnNEUG9tQWIwYVVCZmVFY21hOStvbHMyQ3NDTHNlczFUMGZucDJlVWZPY0tSeU81Cmx5OXFISDBtK2oyT2hnZ0RjOW1JcTUwTUxJMGE1V2g1RjJoOFpkSTlXam1BdzJ5eHRNWEpqV1lQZzVweTRBbEMKcXRvSkVueUNVVHdJdjlPby9EcWZ2Ulhsd1d2bnUvV29HdmlvWlZpeHdqamk1Q2dZN09BQ2tHOWFDZFJlWkZzNApaV014REdzSzdVdzZzLzd2eFppSWphUXhqWXVEQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUUDNtbUF6eFFsdGM3ME10eUNhVjZMVHN4bFdEQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ0dFaW51QWU3VwpkWFBDWkdyNHBpcUNaVGY1enhDbktnWkRXMFVlQ3k5L29UMHlHZEZDTjdtK0wwUnAxb3IzWUozWHc3enRUaG9wCmZyam94ZytRRG1pRk8vd3JWWVgwbDhwbVFWN2czUm9temY1azV5Y24vYUd1SzZLZC9oTTJEQ2JLdnQ3NUUrOU8KMkZnN2RiL0dOMU14Q2QwSzd1Um5kbHdnNHMwcVdVNGZpRGRDWjF0NlMyc2JZTm1zdTJ0c2tGZitpWWoydGVEVgpUbEtaZEpDWGNpYnFjTFBvRHFVbm9kUWZPcXl3cEpnMFFLUWNHQjZhYXRBNUdrcXhVajRuMXgxcDdSaXdHNjJUCml3M1N1SnNEdEtManhFWDQrT0hrQ3pkaXN4ZG9TL2swMmtqTlNFenFPN1RYNVlPNlBCbHhDclVKUzRKVTZabm4KL0NFNEN0dXp6cW1DCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://192.168.1.55:6443
  name: k8s
contexts:
- context:
    cluster: k8s
    user: kelvyn
  name: kelvyn@k8s
current-context: kelvyn@k8s
kind: Config
preferences: {}
users:
- name: kelvyn
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCekNDQWU4Q0ZCRzZla1d2aWwyYlRyRlp6SU1jSVRTN3cwUWJNQTBHQ1NxR1NJYjNEUUVCQ3dVQU1CVXgKRXpBUkJnTlZCQU1UQ210MVltVnlibVYwWlhNd0hoY05NalV3TWpJMU1EYzBNek0xV2hjTk1qWXdNakkxTURjMApNek0xV2pCck1Rc3dDUVlEVlFRR0V3SkRUakVRTUE0R0ExVUVDQXdIUW1WcGFtbHVaekVRTUE0R0ExVUVCd3dIClFtVnBhbWx1WnpFVk1CTUdBMVVFQ2d3TVIwVklaV0ZzZEdoRFlYSmxNUkF3RGdZRFZRUUxEQWRFYVdkcGRHRnMKTVE4d0RRWURWUVFEREFaclpXeDJlVzR3Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLQW9JQgpBUUMxaUZ0cEc4WHVkdVIxMDZRTHNIblBldVI1UXlxLzFCRHhYL1Q2V0VVcU1Eb0h3N1RzTGtCK21CcUs1d3JmCmp1K1ZjYmo0RS9YbnIzdEpLMnMxVTVzbzFpbEhoN0UwdG5sMWd3OTAxNE5mdytteDFsUUZLVUpzN1ZsczZqTmwKVGRZR0REM0E4dHJlN0dhZzVERFVSY2IyRUtkOFZlVjBneWdXMi9kQUlVbWltWHh0aVVYTkF5QVFYbVFBUER3egpReElsTkp1N2p0dTFhSXJ0T2pSK1NZOU9XVEhyUW01RmphMkc1bjdZajkyNWFLQlFMdTArU0ZGWjRpVmJ5TW9lCmJCMG92b2IzSW80Vy95L1c2ZlE0eHBjYXJZWW1yZFNPZGVRY2VwK1JvYXV5bkJ2UVNVWGxkcGhHNTh6d2JuQjQKbjBuenJZeUVnYVR5RDE4UU1yaDRuR2cvQWdNQkFBRXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBSVgwbzhweApYUm41dnNvY2NjeGRrQktOc1h5N0REaXhtbVlXYXhld3RxTXFlcWttR285cGVUSHRRMzVWdVg3dFJwS1JOanRoCko3eEx6U29OMytXemhCY0FBQWlmU1haUFNzd1F1RVA4RmxYcFNiQWM1MzRobVRqTWUxQkIvREowNUtyOStrUUsKQVlUcC9Qd2pmY3JncTlyM1RDYjVHZGpVdnJzRlRkYnlxbTlOMUJTL296WSt0K0ExaXpTaHNHNC9zV0d4bDI3cQp2Z0NNN0hubDN5c3E3aTRkcmpmOFV0WCtPQTRyek4xM3FQcUFIRHNQMWdMcm10V0FKQlF3QjVnUlF6VGJ3RXQzCk55TkxsMkJ4cGc4QzJ2Wm9rb2xZRy91YkNuaCtOWnVaS2J6Z25nTU9EV3dQZjNhb1JCY3JYU1BtQnl4TnJZeHcKeEFWL3o4azhsWWtYTzhFPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBdFloYmFSdkY3bmJrZGRPa0M3QjV6M3JrZVVNcXY5UVE4Vi8wK2xoRktqQTZCOE8wCjdDNUFmcGdhaXVjSzM0N3ZsWEc0K0JQMTU2OTdTU3RyTlZPYktOWXBSNGV4TkxaNWRZTVBkTmVEWDhQcHNkWlUKQlNsQ2JPMVpiT296WlUzV0Jndzl3UExhM3V4bW9PUXcxRVhHOWhDbmZGWGxkSU1vRnR2M1FDRkpvcGw4YllsRgp6UU1nRUY1a0FEdzhNME1TSlRTYnU0N2J0V2lLN1RvMGZrbVBUbGt4NjBKdVJZMnRodVorMkkvZHVXaWdVQzd0ClBraFJXZUlsVzhqS0htd2RLTDZHOXlLT0Z2OHYxdW4wT01hWEdxMkdKcTNVam5Ya0hIcWZrYUdyc3B3YjBFbEYKNVhhWVJ1Zk04RzV3ZUo5Sjg2Mk1oSUdrOGc5ZkVESzRlSnhvUHdJREFRQUJBb0lCQUJ2TEhzTysvdFQ5MndpMwpPSnlaam16WDBmZEc4MXFmYTJDcFltYVo4U3orYVVRYkVLNUFmcHRqU2wwTjlybzN0akxaVUlxYTg4RmZPcTcvCk9ORFhWaUF3ZWUxN3R3UHRGRGVMczJnZVB2MEFqOTBzaFh5c3pvREM3amdndTNHOU14R0Yra1o2YUV4TlFZRk0KcnJVeFliNzIyYzNOa08zL3pybUJRQi9QZU9pdCtUbjRDZUZ0YnF6b3FmMDZwUDNLQmxOaGFJTzA0V2FaV0U4NQpKdjVLM3Jqcm4waWtUVi93dXN1bUVaL3lDT2UvRVJpdnZPWTM3QjJkb0U3WmVrUFovOFhQczBmSUc5VzhuOW5QClRMakRMendSVzVVOUNFQm5xakt6QWkxS0ZSZVBPVEY5eVpJR2ErUHpjOTJxQVNYNEtKaGNGdlJxeG94V25IbGoKVm52Y0ZCRUNnWUVBM0ZWWHFzSTk1RjhjaEx1M2JSZzE3YTA2ZERLMlQ1cGkxeWlDeGxHUTlYR1poT2JZckQvagpxbG1pYlRPbXBGNnlVQjM5dzY0TTIzVDRlbmF1QlRnN09Wdkp3QWw0T3RKcDQySHNzd0JmZlhUd1ZuKzRTMkVVCmtpV3dlSDlxZktBeTZzVEtUWlNEUXVFa01obEx4dkpSaEpFMlA1aEhVM1c5VllGN1p0b0JzTE1DZ1lFQTB1c2EKNXhYUFNHT3BzQ3hEMkwwVjZvU0pWbVpEN2dub3pFdzBwTmFYZ294c21YZWtPOTREaTQrZmVkY0hHcGRJQmdLRApFZ3FTcUFPV0ZveDZHL2V2czBkNytDNmpQZ3Q1Q2wzWVRFcWZJSnNwVWZwU0hHY3lBTktVd2F5RFcxSXZuVTRQCmdFRndUTGFvYVpKQXYwTTR1OVJXOTVmd1d6UUpqeGdJZ1ZtTkdFVUNnWUJaRnJ6YTA2MTQ0S2hFVnk2RWt3eUQKTE03ODJ1QnljV2RUdmhLYW83SnNPK0dxSmprbjlMRldXT1hmSjhwU25lT1ZsM3JiRzA0aGtqdENNU2lOL2IyYwpwS3QvMVpSaW5GK3FUQmNNRGJPT283RG1HTUJvNGprU0d1RXU1NzRqNUJhU2JMMnIvc2ZRUy81NXIxYS9lNDFRCmYvS2laaTA0NXR1R2JsTjZNOTRKRndLQmdGR3J5Z29MTHUySDhmQU80K0tzTFMxWFR0ck8xS1Q2MzFNa2V3b04KTWpQUjdrZHF4WVNORG5CZkY5Q1ZDK0luRERPUGkzTlQ5ci9xUzViRnBJN3AxUFlseXdJcUJQb0VkVVVuVzVjSApHaUVGRS9YemFSSW9mM3RFRDJnRFJnWDVpQWh3YnA0cU9MTHIwOEMxYWk3bGQ3Vjdub1ZYSnpJWnIwM2liNEN1CnpXekZBb0dBRkZ6NGNGaGh0R0xCemtObWFwemYvWkVkN2U1cHJPUzVZNmVxVlJ4ZUFpK1NqdTVBeTdqM0lieEIKWHp1ckhVUzJXTkhEL2JYekZvODJKcUk2RlJCYWFIbGg2eUt4QzdobCtCb210R1NpZm5aWkNNR3p6Y1JTUXdsRgpOS3Fja3B0dk13YXN5RENRakRjOXNPWVI3UHlabGpZM01INXVQR2hOaFoyd25MVHQ3bW89Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```

验证：

```bash
$ kubectl --kubeconfig kelvyn.conf get po
Error from server (Forbidden): pods is forbidden: User "kelvyn" cannot list resource "pods" in API group "" in the namespace "default"
```

# 绑定角色

创建 Role：

这个角色只有 POD 的 get、list、watch 权限

```bash
$ cat << EOF | kubectl apply -f - 
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pods-reader
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
EOF
```

创建 RoleBinding：

```bash
$ cat << EOF | kubectl apply -f - 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: user-kelvyn
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pods-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: kelvyn
EOF
```

验证：

```bash
$ kubectl --kubeconfig kelvyn.conf get po

NAME                              READY   STATUS    RESTARTS   AGE
nginx-deploy-02-5c5c6546d-khhhv   1/1     Running   0          4h51m
nginx-deploy01-8c4b6d777-jv6nw    1/1     Running   0          3d21h
nginx-deploy01-8c4b6d777-zkhbq    1/1     Running   0          3d21h

```

# 删除用户

删除需要的信息及文件：

```bash
# 创建的 Role 及 RoleBinding 需要从 k8s 集群删掉
$ kubectl delete rolebindings user-kelvyn
$ kubectl delete role pods-reader

# 直接删掉创建的文件即可
$ rm -rf /k8s-user/kelvyn
```

# 一键创建及删除脚本

**注意**：默认绑定集群角色：`ClusterRole="cluster-admin"`

```shell
#! /bin/bash
# Author: Kelvyn, Meng
# Blog: https://ivmoe.github.io/
# Date: 2025-02-25
# Modified: 2025-02-25
# Usage: sh k8s_UerAccount_create.sh
# Version: 1.0
# Description: 创建 Kubernetes 用户账户

# 以下变量需要修改，根据实际情况修改
# KUBERNETES_USER: K8S 用户
# USER_CERT_EXPIRE: K8S 用户证书有效期，单位：天
# USER_CONFIG_PATH: K8S 用户配置文件路径
# KUBERNETES_APISERVER: K8S API Server 地址
# KUBERNETES_NAME: K8S集群名称
# KUBERNETES_PKI_PATH: K8S PKI 证书路径

KUBERNETES_USER="kelvyn"
USER_CERT_EXPIRE="365"
USER_CONFIG_PATH="/k8s-user"
KUBERNETES_NAME="k8s"
KUBERNETES_APISERVER="https://192.168.1.55:6443"
KUBERNETES_PKI_PATH="/etc/kubernetes/pki"

# 以下变量无需修改，或者根据实际情况修改
KUBERNETES_CA_PATH="${KUBERNETES_PKI_PATH}/ca.crt"
KUBERNETES_CA_KEY_PATH="${KUBERNETES_PKI_PATH}/ca.key"
USER_KEY_FILE="${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.key"
USER_CSR_FILE="${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.csr"
USER_CERT_FILE="${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.crt"


CREATE_USER_CONFIG() {
    echo "-----> INFO: 创建用户配置文件"

    if [ ! -e ${USER_CONFIG_PATH}/${KUBERNETES_USER} ]; then
        mkdir -p ${USER_CONFIG_PATH}/kelvyn
    fi

    # 1. 创建私钥
    if [[ ! $(type openssl) ]]; then
        echo "-----> ERROR: openssl 工具未安装, 请安装后继续"
        echo "-----> INFO: Debian 系: apt install -y openssl"
        echo "-----> INFO: RedHat 系: yum install -y openssl 或 dnf install -y openssl"
        exit 1
    fi
    openssl genrsa -out ${USER_KEY_FILE} 2048

    # 2. 创建证书请求
    openssl req -new -key ${USER_KEY_FILE} -out ${USER_CSR_FILE} -subj "/CN=${KUBERNETES_USER}/O=system:masters"

    # 3. 生成证书
    openssl x509 -req \
        -in ${USER_CSR_FILE} \
        -CA ${KUBERNETES_CA_PATH} \
        -CAkey ${KUBERNETES_CA_KEY_PATH} \
        -CAcreateserial \
        -out ${USER_CERT_FILE} -days ${USER_CERT_EXPIRE}

    if $? -ne 0 ; then
        echo "-----> ERROR: 生成证书失败!"
        exit 1
    fi

    # 4. 创建 kubeconfig 文件
    kubectl config set-cluster ${KUBERNETES_NAME} \
        --certificate-authority=${KUBERNETES_CA_PATH} \
        --embed-certs=true \
        --server=${KUBERNETES_APISERVER} \
        --kubeconfig=${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.kubeconfig

    # 5. 设置客户端认证
    kubectl config set-credentials ${KUBERNETES_USER} \
        --client-certificate=${USER_CERT_FILE} \
        --client-key=${USER_KEY_FILE} \
        --embed-certs=true \
        --kubeconfig=${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.kubeconfig

    # 6. 设置上下文 Conetxt
    kubectl config set-context ${KUBERNETES_USER}@${KUBERNETES_NAME} \
        --cluster=${KUBERNETES_NAME} \
        --user=${KUBERNETES_USER} \
        --kubeconfig=${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.kubeconfig

    # 7. 设置默认上下文
    kubectl config use-context ${KUBERNETES_USER}@${KUBERNETES_NAME} \
        --kubeconfig=${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.kubeconfig
    
    if $? -ne 0 ; then
        echo "-----> ERROR: 生成证书失败!"
        exit 1
    fi

}

BIND_ROLE() {
    echo "-----> INFO: 绑定 Kubernetes User 到 ClusterRole.cluster-admin 角色"
    
    kubectl create clusterrolebinding ${KUBERNETES_USER}-cluster-admin-binding \
        --clusterrole=cluster-admin \
        --user=${KUBERNETES_USER}

#     cat << EOF | kubectl apply -f - 
# apiVersion: rbac.authorization.k8s.io/v1
# kind: ClusterRoleBinding
# metadata:
#   name: ${KUBERNETES_USER}-cluster-admin-binding
# roleRef:
#   apiGroup: rbac.authorization.k8s.io
#   kind: ClusterRole
#   name: cluster-admin
# subjects:
# - apiGroup: rbac.authorization.k8s.io
#   kind: User
#   name: ${KUBERNETES_USER}
# EOF
}

ENDING() {
    if [[ $? -eq 0 ]]; then
        echo
        echo "############################################################################################################################"
        echo
        echo "-----> INFO: Kubernetes User 创建完成!"
        echo "-----> Kubernetes User: ${KUBERNETES_USER}"
        echo "-----> K8S 用户有效期：${USER_CERT_EXPIRE} 天"
        echo "-----> kubeconfig 文件路径：${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.kubeconfig"
        echo "-----> 登录集群两种办法："
        echo "----->   方式一：切换当前上下文"
        echo "          $ kubectl config use-context ${KUBERNETES_USER}@${KUBERNETES_NAME} --kubeconfig=${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.kubeconfig"
        echo "----->   方式二：执行 kubectl 命令时, 指定 kubeconfig 文件路径"
        echo "          $ kubectl CMD --kubeconfig=${USER_CONFIG_PATH}/${KUBERNETES_USER}/${KUBERNETES_USER}.kubeconfig"
        echo "-----> 创建 Kubernetes User 脚本执行完毕!"
        exit 0
    else
        echo "-----> Kubernetes User 创建失败!"
        exit 1
    fi
    
}

DELETE_USER() {
    echo "-----> INFO: 删除集群中的 RoleBinding"
    kubectl delete clusterrolebinding ${KUBERNETES_USER}-cluster-admin-binding

    echo "-----> WARNING: 删除用户配置文件"
    # 检查 USER_CONFIG_PATH 和 KUBERNETES_USER 是否为空
    if [[ -z "${USER_CONFIG_PATH}" || -z "${KUBERNETES_USER}" ]]; then
        echo "-----> ERROR: 变量 USER_CONFIG_PATH 或 KUBERNETES_USER 未设置或为空"
        exit 1
    fi

    # 使用 ${var:?} 确保路径不为空
    rm -rf "${USER_CONFIG_PATH:?}/${KUBERNETES_USER:?}"
    echo "-----> INFO: 删除用户配置文件成功!"
    echo "-----> INFO: 集群用户 ${KUBERNETES_USER} 删除成功!"
    echo "-----> INFO: 删除 Kubernetes User 脚本执行完毕!"
    exit 0
}

main() {
    echo "###### Date: $(date) ######"
    read -rp '-----> INFO: 创建用户"1", 删除用户"2": ' answer
    echo
    case ${answer} in
        1)
            CREATE_USER_CONFIG
            BIND_ROLE
            ENDING
            ;;
        2)
            DELETE_USER
            ;;
        *)
            echo "----->ERROR: 请输入正确的选项: 1 或 2"
            exit 1
            ;;
    esac
    
}

main | tee -a /tmp/k8s_user_create.log
```

脚本执行，创建和删除的截图。
[![pE1Tlff.md.png](https://s21.ax1x.com/2025/02/25/pE1Tlff.md.png)](https://imgse.com/i/pE1Tlff)


脚本较为简单，有很多没有考虑到的地方，如果有需求可以提，我来修改和补充！