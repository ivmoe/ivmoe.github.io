---
title: Ansible—authorized_key模块
categories:
  - 笔记
tags:
  - Ansible
abbrlink: 53862
date: 2024-11-24 22:18:33
updated: 
sticky: 
---

> ansible.posix.authorized_key 模块 – 新增或删除 SSH 授权密钥
> 官方文档：https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html


用来配置密钥实现免密码登录。

# 常规做法

```bash
# 生成密钥对
ssh-keygen -t rsa -C kelvyn@gehealthcare.com

# 复制公钥到目标服务器
ssh-copy-id -i id_rsa.pub root@192.168.1.100

# 查看目标服务器的公钥
cat /root/.ssh/authorized_keys
```

# 使用ansible为多台目标主机添加公钥

创建密钥对：

```bash
# 生成密钥对
ssh-keygen -t rsa -C kelvyn@gehealthcare.com

# 暂且认为生成的文件为 /root/id_rsd.pub
```

创建playbook.yaml：

```yaml
- name: Add public key to multiple hosts
  hosts: all

  tasks:
    authorized_key:
      user: root
      state: present
      key: "{{ lookup('file', '/root/id_rsa.pub') }}"
```

```bash
ansible-playbook playbook.yaml
```

## 注意：

第一次连接主机会验证指纹，需要输入yes进行确认。

我们可以修改ansible的配置文件`/root/ansible/ansible.cfg`

```yaml
# 取消注释，或者直接添加这一行
host_key_checking = False
```

## 官网添加配置示例：

```yaml
- name: Set authorized key taken from file   从一个文件中添加授权密钥
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: "{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"

- name: Set authorized keys taken from url    从一个url中添加授权密钥
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: https://github.com/charlie.keys

- name: Set authorized keys taken from url using lookup    使用lookup方法从一个url中添加授权密钥
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: "{{ lookup('url', 'https://github.com/charlie.keys', split_lines=False) }}"

- name: Set authorized key in alternate location    添加授权密钥到其他文件（此处指/home/kelvyn/.ssh/authorized_keys以外的其他地方）
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: "{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
    path: /etc/ssh/authorized_keys/charlie
    manage_dir: false

- name: Set up multiple authorized keys    添加多个授权密钥（这个超级有用呦~~~），需要注意的是，当使用with_*的时候，每次迭代都会进行移除操作（exclusive）。
  ansible.posix.authorized_key:
    user: deploy
    state: present
    key: '{{ item }}'
  with_file:
    - public_keys/doe-jane
    - public_keys/doe-john

- name: Set authorized key defining key options    设置授权密钥并定义密钥选项（key_options：附加到密钥中的字符串）
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: "{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
    key_options: 'no-port-forwarding,from="10.0.1.1"'

- name: Set authorized key without validating the TLS/SSL certificates   从url中设置，无需校验SSL证书（在你绝对信任对方url的情况下使用）
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: https://github.com/user.keys
    validate_certs: false

- name: Set authorized key, removing all the authorized keys already set    添加即将要设置的密钥，而且删除所有已经存在的密钥
  ansible.posix.authorized_key:
    user: root
    key: "{{ lookup('file', 'public_keys/doe-jane') }}"
    state: present
    exclusive: true

- name: Set authorized key for user ubuntu copying it from current user    设置正在登录的主控机用户的公钥到目标服务器的ubuntu用户下
  ansible.posix.authorized_key:
    user: ubuntu
    state: present
    key: "{{ lookup('file', lookup('env','HOME') + '/.ssh/id_rsa.pub') }}"
```