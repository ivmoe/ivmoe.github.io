---
title: 自动化运维工具——Ansible Playbook
categories:
  - 笔记
tags:
  - Ansible
abbrlink: 46999
date: 2024-11-24 22:18:33
updated: 2024-12-2 14:59:12
sticky: 
---

> 官方文档：[https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html)

### 📜1. 什么是playbook

中文名：剧本，它是一个自动化处理脚本。 Playbook采用YAML语言编写。

* * *

### 📜2. playbook演示

以下做个简单的操作演示，编写好主机清单后再进行剧本的编写。

（所以说嘛，基础很重要，如果不记得主机清单[https://blog.csdn.net/qq_41765918/article/details/121676991](https://blog.csdn.net/qq_41765918/article/details/121676991)和配置文件[https://blog.csdn.net/qq_41765918/article/details/121706648](https://blog.csdn.net/qq_41765918/article/details/121706648)是如何运用的，快快去学习。）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b7b8fe21006be0d231e701c4c17388e.png)

```yaml
[student@servera ~]$ cat hosts 
servera

[student@servera ~]$ cat webserver.yml 
---
- name: play to setup web server
  hosts: servera
  remote_user: root
  become: yes
  become_method: sudo
  tasks:
  - name: latest httpd version install
    yum:
      name: httpd
      state: latest

[student@servera ~]$ ansible-playbook -i hosts webserver.yml 
PLAY [play to setup web server] *********************************************************

TASK [Gathering Facts] ******************************************************************
ok: [servera]

TASK [latest httpd version install]******************************************************
changed: [servera]

PLAY RECAP ******************************************************************************
servera                    : ok=2    changed=1    unreachable=0    failed=0
```

* * *

### 📜3. Playbook工作流程

![](https://i-blog.csdnimg.cn/blog_migrate/fa2401007dfe0c6653523503074cc279.png)

+   playbook 剧本是由一个或多个"play"组成的列表
+   play的主要功能在于将预定义的一组主机，装扮成事先通过ansible中的task定义好的角色。Task  
    实际是调用ansible的一个module，将多个play组织在一个playbook中，即可以让它们联合起  
    来，按事先编排的机制执行预定义的动作。
+   Playbook 文件是采用YAML语言编写的

（就正如图片小人书，剧本都是由我们根据特定需求编写好的“脚本”，此“脚本”运用各种模块通过作用于特定的主机清单而达到需求。）

![image-20211205170916711](https://i-blog.csdnimg.cn/blog_migrate/c77563b1a955b7be46fa05a967cfb826.png)

* * *

### 📜4. YAML语法简介

这里只涉及到playbook相关的语法（更多请参考官网http://www.yaml.org）。

**语法非常严格，请仔细仔细再仔细。**

```yaml
在单一档案中，可用连续三个连字号(---)区分多个档案。另外，还有选择性的连续三个点号( ... )用来表示档案结尾
 次行开始正常写Playbook的内容，一般建议写明该Playbook的功能
• 使用#号注释代码
• 缩进必须是统一的，不能空格和tab混用,一般缩进2个空格（可以改造tab为缩进）
• 缩进的级别也必须是一致的，同样的缩进代表同样的级别，程序判别配置的级别是通过缩进结合换行来实现的
• YAML文件内容和Linux系统大小写判断方式保持一致，是区别大小写的，key/value的值均需大小写敏感
• key/value的值可以写在同一行，也可换行写。同一行使用 , 逗号分隔
• value可是个字符串，也可是另一个列表
• 一个完整的代码块功能需最少元素需包括 name和task
• 一个name只能包括一个task
• 使用 | 和 > 来分隔多行，实际上这只是一行。
        include_newlines: |
            exactly as you see
            will appear these three
            lines of poetry

        ignore_newlines: >
            this is really a
            single line of text
            despite appearances
• Yaml中不允许在双引号中出现转义符号，所以都是以单引号来避免转义符错误 
• YAML文件扩展名通常为yml或yaml
```

很多刚学习的同学，往往运行报错就挂在语法上，后续可通过不断的练习来理解。

（语法基础真的很重要，别说了你又不听，听了你又不懂，懂了你又不做，做了你又做错，错了你又不认，认了你又不改，改了你又不服。）

![image-20211205171204735](https://i-blog.csdnimg.cn/blog_migrate/a8b2db17d6763604352a30076c82faaa.png)

* * *

#### 📑List：列表

其所有元素均使用"-"打头

```yaml
- web
- dns
-空格web        # 书写格式
```

* * *

#### 📑Dictionary：字典(键值对)

通常由多个key与value构成

```yaml
多行写法：
name: hunk
blog: "xxxxx"
name:空格hunk   > 这个冒号后面必须是一个空格

同一行写法：
需要使用{ }
{name: hunk, blog: "xxxxxx"}  > 逗号后建议使用留一个空格

布尔值的表示法：
yes/no  true/false

create_key: yes
needs_agent: no
knows_oop: True
likes_emacs: TRUE
uses_cvs: false
```

* * *

### 📜5. Playbook核心元素

#### 📑name

可选配置项，可有助于记录playbook的运行说明。

* * *

#### 📑hosts

hosts 行的内容是一个或多个组或主机的 patterns,以逗号为分隔符。通常是/etc/ansible/hosts定义的主机列表

remote_user 就是远程执行任务的账户名:

```yaml
---
- hosts: web,dns
  remote_user: root
```

* * *

#### 📑tasks

任务集

```yaml
  tasks:
    - name: install httpd
      yum: 
      name: httpd

    - name: start httpd
      service: name=httpd state=started
```

* * *

### 📜6. 改造vi中tab键功能

为了能更加轻松的编辑playbook，可设置在vi编辑yaml文件时，按下tab键会进行一个双空格缩进。

在 **$HOME/.vimrc** 添加以下内容

```shell
autocmd FileType yaml setlocal ai ts=2 sw=2 et
```

参数解释：

```shell
set ai    # 自动缩进
set ts=2  # tabstop，表示按一个tab之后，显示出来的相当于几个空格，默认的是8个。
set sw=2  # shiftwidth，表示每一级缩进的长度
set et    # expandtab，将tab转成空格，缩进用空格来表示
```

（别问我为什么要改写，因为当你见到别人噼里啪啦地敲完键盘写好playbook时，你可能还在默念按了多少个空格。>.<）
![](https://i-blog.csdnimg.cn/blog_migrate/8a46427eae9e18b6877ab760bd957d56.png)

* * *

### 📜7. playbook书写风格

以下书写例子中，就是shorthand格式，比较旧的书写方法：

```yaml
- name: copy new yum config to host
  copy: src=/etc/yum.repos.d/ dest=/etc/yum.repos.d/
```

普遍的yaml格式书写：

```yaml
- name: copy new yum config to host
  copy: 
    src: /etc/yum.repos.d/ 
    dest: /etc/yum.repos.d/
```

基本都是使用新的写法，便于阅读和排错。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/387481714c72c5914c61a7f3472e4713.png)

* * *

### 📜8. playbook执行

**检测语法：ansible-playbook --syntax-check webserver.yml**

**模拟执行：ansible-playbook -C webserver.yml**

**真实执行：ansible-playbook webserver.yml**

```shell
执行的详细程度：
-v   ：显示任务结果。
-vv  ：任务结果和任务配置都会显示
-vvv ：包含关于与受管主机连接的信息
-vvvv：增加了连接插件相关的额外详细程度选项，包括受管主机上用于执行脚本的用户，以及所执行的脚本
```

* * *

### 📜9. 练习演示----如何编写playbook

##### 📑设置tab键缩进

```shell
[student@servera ~]$ cat .vimrc 
autocmd FileType yaml setlocal ai ts=2 sw=2 et
```

* * *

##### 📑查看配置文件和主机清单

```shell
[student@servera ~]$ mkdir playbook

[student@servera playbook]$ cat ansible.cfg 
[defaults]
inventory=inventory
remote_user=student

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False

[student@servera playbook]$ cat inventory 
[web]
serverc
serverd
```

* * *

##### 📑编写所需要的剧本

```yaml
[student@servera playbook]$ cat site.yml 
---
- name: Install and start Aapche HTTPD
  hosts: web
  tasks:
    - name: httpd package is present
      yum:
        name: httpd
        state: present

    - name: correct index.html is present
      copy:
        content: "This is a test page.\n"
        dest: /var/www/html/index.html

    - name: httpd is started
      service:
        name: httpd
        state: started
        enabled: true
```

* * *

##### 📑语法检查

```shell
[student@servera playbook]$ ansible-playbook --syntax-check site.yml 

playbook: site.yml
```

* * *

**运行剧本前进行语法检查是一个良好习惯。**

##### 📑执行剧本

```shell
[student@servera playbook]$ ansible-playbook site.yml
PLAY [Install and start Aapche HTTPD] ***************************************************
TASK [Gathering Facts] ******************************************************************
ok: [serverc]
ok: [serverd]

TASK [httpd package is present] *********************************************************
changed: [serverd]
changed: [serverc]

TASK [correct index.html is present] ****************************************************
changed: [serverc]
changed: [serverd]

TASK [httpd is started] *****************************************************************
changed: [serverc]
changed: [serverd]

PLAY RECAP ******************************************************************************
serverc    : ok=4    changed=3    unreachable=0    failed=0   
serverd    : ok=4    changed=3    unreachable=0    failed=0
```

* * *

##### 📑幂等性

幂等性这一概念源于数学，原意是指一个操作如果连续执行多次所产生的结果与仅执行一次的效果相同，那么我们就称这个操作是幂等的。

```shell
[student@servera playbook]$ ansible-playbook site.yml
…………
```

此为Ansible重要特性之一，用练习演示更容易进行理解。

* * *

##### 📑结果访问检查

```shell
[student@servera playbook]$ curl serverc
This is a test page.
[student@servera playbook]$ curl serverd
This is a test page.
```

* * *

### 📜10. playbook提权

可以写在剧本中，与hosts，tasks同级

```yaml
- hosts: all
  become: true
  become_method: sudo
  become_user: root
  
  tasks:
  - debug:
     msg: "Test"
```

当然，不在剧本上编写，在配置文件上编写也可以，如果两个地方都进行编写，在剧本中的优先级会比配置文件上的高，例如：配置文件上启动提权become: yes，但剧本上配置了become: no，最终生效为剧本上的配置。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4a6e279fcb1b9662c098d3cd667b6cb4.png)

* * *

### 📜11. 善于模块文档

**ansible-doc -l 查看列表**

**ansible-doc modulename 查看模块帮助文件**

（上一篇文章总结处[https://blog.csdn.net/qq_41765918/article/details/121722471](https://blog.csdn.net/qq_41765918/article/details/121722471)提及，查看帮助真的很重要~~）

![image-20211204212605255](https://i-blog.csdnimg.cn/blog_migrate/003a55e78db200e79553b22e2bce90f5.png)

* * *

### 📜12. 练习演示----执行多个playbook

##### 📑查看配置文件和主机清单

```shell
[student@servera ~]$ mkdir playbook-multi
[student@servera ~]$ cd playbook-multi/
[student@servera playbook-multi]$ cat ansible.cfg 
[defaults]
inventory=inventory
remote_user=student

[privilege_escalation]
become=False
become_method=sudo
become_user=root
become_ask_pass=False

[student@servera playbook-multi]$ cat inventory 
servera
```

* * *

##### 📑编写所需要的剧本

```yaml
[student@servera playbook-multi]$ cat web.yml 
---
- name: Enable web services
  hosts: servera
  become: yes
  tasks:
    - name: latest version of httpd and firewalld installed
      yum:
        name:
          - httpd
          - firewalld
        state: latest

    - name: test html page is installed
      copy:
        content: "Hello World!\n"
        dest: /var/www/html/index.html

    - name: firewalld enabled and running
      service:
        name: firewalld
        enabled: true
        state: started

    - name: firewalld permits http service
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes

    - name: httpd enabled and running
      service:
        name: httpd
        enabled: true
        state: started

- name: Test web server
  hosts: localhost
  become: no
  tasks:
    - name: connect to web server
      uri:
        url: http://servera
        return_content: yes
        status_code: 200
```

* * *

##### 📑语法检查

```shell
[student@servera playbook-multi]$ ansible-playbook --syntax-check web.yml 

playbook: web.yml
```

* * *

##### 📑使用-v执行剧本

```shell
[student@servera playbook-multi]$ ansible-playbook web.yml -v
PLAY [Enable web services] **************************************************************

TASK [Gathering Facts] ******************************************************************
ok: [servera]

TASK [latest version of httpd and firewalld installed] ******************************************
ok: [servera] => {"changed": false, "msg": "", "rc": 0, "results": ["All packages providing httpd are up to date", "All packages providing firewalld are up to date", ""]}

…………
TASK [connect to web server] ************************************************************
ok: [localhost] => {"accept_ranges": "bytes", "changed": false, "connection": "close", "content": "Hello World!\n", "content_length": "37", "content_type": "text/html; charset=UTF-8", "cookies": {}, "cookies_string": "", "date": "Fri, 04 Sep 2020 12:54:49 GMT", "etag": "\"25-5ae7c5e78c9df\"", "last_modified": "Fri, 04 Sep 2020 12:54:27 GMT", "msg": "OK (37 bytes)", "redirected": false, "server": "Apache/2.4.6 (Red Hat Enterprise Linux)", "status": 200, "url": "http://servera"}

PLAY RECAP **********************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0   
servera    : ok=6    changed=0    unreachable=0    failed=0
```

* * *

### 💡总结

+   yaml语法非常严格，仔细编写。
+   改造tab键以提高编写剧本的效率。
+   语法检查是一个良好习惯。
+   善用模块文档查看参数说明与例子。
+   通过练习与实验了解幂等性。
+   可使用 -v 参数查看详细输出进行排错。


> 转载自： IT民工金鱼哥 [https://blog.csdn.net/qq_41765918/article/details/121732601](https://blog.csdn.net/qq_41765918/article/details/121732601)，如有侵权，请联系删除。
>
> 虽然文章是转载的，我转载这一篇是因为他说了几个其他教程都没有的地方，但还是太太太基础了，有些地方没有讲到，比如变量传递、when、template模块、handlers等等的使用，所以我还推荐另一篇文章：[https://www.cnblogs.com/yanjieli/p/10969299.html](https://www.cnblogs.com/yanjieli/p/10969299.html)
