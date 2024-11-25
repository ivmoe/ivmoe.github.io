---
title: 自动化运维工具——Ansible Roles
categories:
  - 笔记
tags:
  - Ansible
abbrlink: 
date: 2024-11-25 14:10:49
updated: 
sticky: 
---

{% note warning modern %}
注意：学习ansible的roles前，请一定先学习playbook！！！
注意：学习ansible的roles前，请一定先学习playbook！！！
注意：学习ansible的roles前，请一定先学习playbook！！！
{% endnote %}


> 官方文档：https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html

# 角色目录树：

```bash
roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
```

以上，是咱们再工作学习过程中较为通用并且使用频率最高的，官网介绍的其他目录可以通过官网等渠道自行学习，目的是一样的！

# 示例

我们以创建一个部署nginx的roles为例：

## 创建roles目录：

```bash
mkdir -p roles/<角色名>/{tasks, handlers, templates, files, vars}
```

## 创建各个文件：

`nginx/tasks/main.yaml`文件

```yaml
- name: Add group www
  group:
    name: '{{ user_group }}'
    gid: 1000

- name: Add user www
  user:
    name: '{{ user_group }}'
    group: '{{ user_group }}'
    nocreatehome: yes
    shell: /sbin/nologin

- name: Install package
  yum:
    name: nginx
    state: present

- name: template nginx.conf.j2
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx_conf
  notify:
    - Restart nginx

- name: Start Nginx
  systemd:
    name: nginx
    state: started
    enabled: yes

```

`nginx/handlers/main.yaml`文件

```yaml
- name: Restart nginx
  systemd:
    name: nginx
    state: restarted
```

`nginx/templates/nginx.conf.j2`文件(这个配置文件不全，照抄是不对的，仅用作示例)

```jinja2
server {
    listen       {{ nginx_port }};
    server_name  {{ ansible_hostname }};  // ansible_hostname, 这是ansible内置的变量
    }
```

`nginx/vars/main.yaml`文件

```yaml
user_group: www
nginx_port: 80
```

playbook的入口文件`roles_nginx.yaml`(与roles在同一个目录)：

```yaml
- hosts: web
  remote_user: root
  roles:
    - nginx
```

## 部署

使用命令执行部署就可以了：

```bash
ansible-playbook roles_nginx.yaml
```