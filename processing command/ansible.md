# ansible
>开源自动化平台
>类型：控制节点（ansible 服务器）和受管主机。
>所有受管主机都要在清单内

## 基础命令

>查看模块帮助
```
ansible-doc -l 查看ansible当前支持的所有模块
ansible-doc -s 查看指定模块的使用说明，比如ansible-doc -s yum
```
>ansible命令应用基础：
>>语法：ansible <host-pattern> [-f forks] [-m module_name] [-a args]
```
-f  forks：启动的并发线程数;
-m module name：是使用的模块
-a agrs：模块特定的参数
```
>>例：ansible 192.168.98.44 -m command -a 'date'



>运维工具分类：
```
	agent：puppet,func
	agentless：ansible，fabric，ssh service
	
	ansible核心组件：
	ansible core
	host iventory
	core modules
	custom modules
	playbooks（yaml文件）
	connect plugin
```

## ansible特性：
>基于python语言实现，由Paramiki，PyYAML和Jinjia2三个关键模块

>部署简单，agentless
>默认使用SSH协议

>主从模式
```
master:ansible,ssh client
slave:ssh server
```

>支持自定义模块：支持各种编程语言
>支持playbook
>基于“模块”完成各种“任务”



## ansible安装
>安装
```
yum list all *ansible
yum info ansible
```
### 配置ssh登录环境

>基于密钥登录
>在inventory文件中指定帐号密码

### ansible 参数：
>-m 指定模块
>-h 指定主机

>安装：
```
yum install ansible -y
rpm -ql ansible
```
>重要文件：
>>主配置文件：`/etc/ansible/ansible.cfg`
>>远程主机信息文件：`inventory：/etc/ansible/hosts`

>默认登录方式：
```
[web]
192.168.98.44 ansible_ssh_pass=1 ansible_ssh_user=root

```

### 基于密钥登录

```
cd /etc/ansible
vim hosts
[webserver]
192.168.98.44
```

### 设置密钥：
```
ssh-keygen -t rsa
ssh-copy-id -i /root/.ssh/名称.pub root@192.168.98.44
```
>测试免密登录：`ssh 192.168.98.44`



## 常用模块：
>command： 命令模块（默认模块）,用于在远程执行命令；
>>例：ansible all -a 'date'；

>cron：
```
     state:状态
	present：安装
	absent：移除
```
>>例：让远程主机每10分钟执行一次“输出hello world”；

`ansible 192.168.98.44 -m cron -a 'minute="*/10" job="/bin/echo hello" name="test cron job"'`

>检查远程主机：
`ansible 192.168.98.44 -a 'crontab -l'`

>移除任务：
`ansible 192.168.98.44 -m cron -a 'minute="*/10" job="/bin/echo hello" name="test cron job" state=absent'`

### user： 用户模块；
>例：
>>添加用户：`ansible -m user -a 'name="user1"'`；
>>删除用户：`ansible -m user -a 'name="user1" state=absent'`；
>>创建组：`ansible all -m group -a 'name=mysql gid=306 system=yes'`；


### copy：复制模块:
```
	src:指定本地源文件路径；
	dest：指定远程目标文件复制路径；
	content：取代src参数，指定源文件内容；
```
>>例：
```

	ansible all -m copy -a 'src=/etc/fstab dest=/tmp/fatab.ansible owner=root mode=640

	ansible all -m copy -a 'content="hello world\nMy name is leon" dest=/tmp/test.ansible'

```

### file：文件管理模块
```
path：指定文件路径;
	
创建链接文件：
	state=link：创建链接文件；
	src=指明源文件；
	path：指明链接文件路径；

```

>例：
```
	ansible all -m file -a 'owner=mysql group=msyql mode=644 /tmp/fatab.ansible

	ansible all -m file -a 'path=/tmp/fstab.link src=/tmp/fatab.ansible state=link'
```

### ping 网络测试模块

>例：ansible all -m ping 


### service 服务管理模块:
```
	enabled：是否开机启动，true或false；
	name：服务名；
	state：状态，started/stopped/restarted；

```
>检查远程主机服务状态：
```
	ansible all -a 'service httpd status'
	ansible all -a 'chkconfig --list httpd'

```
>起服务，加开机自启
`ansible all -m service -a 'enabled=true name=httpd state=started'`

### shell  变量模块（command不支持变量参数）
```

	ansible all -m user -a 'name=user1'
	尝试：ansible all -m command -a 'echo 123456|passwd --stidn user1'
	ansible all -m shell -a 'echo 123456|passwd --stidn user1'

```

### script 脚本模块；

>注意：基于相对路径方式使用；
>创建一个本地脚本:
```
	vim /tmp/test.sh
	#!/bin/bash
		echo "hello ansible" > /tmp/script.ansible
		useradd user2
		chmod +x /tmp/test.sh

	cd /tmp
		ansible all -m script -a "test.sh"

```

### yum 安装程序包模块；
```
	name:指明程序包的名称；
	state：present/lastest安装，absent卸载；

	ansible all -m yum -a "name=httpd"
```
>查询 `rpm -q httpd`

>卸载：
`ansible all -m yum -a "name=httpd state=absent"`


>setup 收集远程主机可用的facts（相关信息，如）；

>>每个被管理节点在接收并运行管理命令之前，会将自己主机相关信息，如操作系统版本，IP地址等报告给远程的ansible主机；


>>例：ansbile all -m setup（所有主机的所有信息）；



## playbook：剧本

> yaml文件：*.yaml，*.yml，

```
	序列：
	
	- 我是...........................
	-  他是.....................，他有以下特点：
	    - 1、...........
	    - 2、.............
```
>标准剧本规范：vim test.yaml或vim test.yml
```
	---
	- name: chuangjian zu and yonghu
	  hosts: web
	  tasks:
	    - name: create group
	      group:
	        name: abc
	        state: present
	    - name: create user
	      user:
	        name: andy
	        group: abc
	...
	
	
	- hosts: web
	  gather_facts: no
	  tasks:
	    - name: create group
	      group:
	        name: abc
	        state: present
	    - name: create user
	      user:
	        name: andy
	        group: abc
	
	
	ansible-playbook test.yml
```

### 案例一

>编写一个剧本，为webs组内主机安装http，并开启服务，加入开机，开启防火墙，放行http服务;

>命令：

```
	ansible webs -m yum -a 'name=httpd'
	ansible webs -m service -a 'name=httpd state=started enabled=true'
	ansible webs -m firewalld -a 'service=http permanent=true immediate=true state=enabled'

```

> 剧本:
```
---
- hosts: webs
  gather_facts: no
  tasks:
    - name: install httpd package
      yum:
        name: httpd
    - name: start httpd and enable
      service:
        name: httpd
        state: started
        enabled: true
    - name: start firewalld
      service:
        name: firewalld
        state: started
    - name: setting firewalld for http
      firewalld:
        service: http
        permanent: true
        immediate: true
        enabled: true
```

### 案例二
>为服务器组web的组内主机安装httpd，并启动httpd，加入开机自启
>为服务器组db的组内主机新建系统管理员dbadmin
>剧本：
```
	- hosts: web
	  tasks:
	    - name: install httpd 
	      yum:
	        name: httpd
	    - name: start httpd
	      service:
	        name: httpd
	        state: started
	        enabled: true
	
	- hosts: db
	  tasks:
	    - name: create user
	      user:
	        name: dbadmin
	        group: root

```

## 用户管理

> 非管理员的ansible工作相关：
>> 需要配置非管理员自己的ansible.cfg配置文件
```
mkdir my-ansible
cd my-ansible
vim ansible.cfg
[defaults]
inventory = /home/leon/my-ansible/inventory 或 ./inventory
remote_user = aa

[privilege_escalation]
become=true
become_method=sudo
become_user=root
become_ask_pass=False
```

> 配置leon用户自己的清单 
```
vim /home/leon/my-ansible/inventory

[web]
192.168.43.66 ansible_become_method=su ansible_become_user=root ansible_become_pass=1
[db]
192.168.43.110 ansible_become_method=su ansible_become_user=root ansible_become_pass=1
```
>重做免密（当前非管理员用户的登录免密）
```
ssh-keygen
ssh-copy-id -i aa@web01
ssh-copy-id -i aa@db01
```

>测试：ansible all -m ping 


### 在清单定义变量

>案例：
>>为web组主机修改http端口为8080
>>为db组主机修改http端口为888
>>使用template模块：
>>template：在复制文件过程中，该文件可以使用变量
>>区别copy：不能使用变量

>>template文件为jinja2文件，命名一般为*.j2:`vim httpd.conf.j2`

>>关闭防火墙
```
	- hosts: all
	  tasks:
	  - name: copy httpd.conf.j2 to web server
	    template:
	      src: httpd.conf.j2
	      dest: /etc/httpd/conf/httpd.conf
	  - name: restart httpd
	    service:
	      name: httpd
	      state: restarted
```

### 在剧本定义变量
```
- hosts: all
  vars:
    a: leon
    b: root
  tasks:
  - name: create user
    user:
      name: {{ a }}
      group: {{ b }}
```

>案例：
```
register：自定义（注册变量）

- hosts: all
  tasks:
    - name: test register
      shell: hostname
      register: d
    - name: display db group hostname
      debug: msg="{{ d.stdout }}"
```

### 清空源目录剧本：
```
	- hosts: all
	  gather_facts: no
	  tasks:
	    - name: list all /etc/yum.repos.d/
	      shell: ls /etc/yum.repos.d
	      register: d
	    - name: delete all file from /etc/yum.repos.d
	      file: path=/etc/yum.repos.d/{{ item }} state=absent
	      with_items:
	        - "{{ d.stdout_lines }}"
```

> ansible all -m setup（事实facts）

> 条件判断when

>> 为所有红帽和乌班图系统的服务器安装elinks

```
	- hosts: all
	  vars:
	    aa:
	      - Redhat
	      - ubuntu
	  tasks:
	    - name: install elinks package
	      yum:
	        name: elinks
	        state: present
	      when: ansible_distribution in aa

```

## 触发器（处理程序）handlers
### 案例：
>编写一个playbook，按要求使用template模块修改以下主机http端口，并实现访问

* 1.修改web01的http端口为8080
* 2.修改db01的http端口为888
* 3.在防火墙开启的状态下，能够访问http://web01:666
* 4.在防火墙开启的状态下，能够访问http://db01:888

>步骤：
* 1.制作模板：httpd.conf.j2,将42行修改为：Listen {{ http_port }}
* 2.配置清单，设置http_port变量,web01的配置: http_port=666 ,db01的配置：http_port=888 



>编写剧本：
- hosts: all
  tasks:
    - name: start firewalld for all servers
      service:
        name: firewalld
        state: started
        enabled: true
      tags: a1
    - name: set http service for firewalld
      firewalld:
        service: http
        permanent: true
        immediate: true
        state: enabled
      tags: a2
    - name: set port 666 for firewalld
      firewalld:
        port: 8080/tcp
        permanent: true
        immediate: true
        state: enabled
      when: ansible_hostname == 'web01'
      tags: a3
    - name: set port 888 for firewalld
      firewalld:
        port: 888/tcp
        permanent: true
        immediate: true
        state: enabled
      when: ansible_hostname == 'db01'
      tags: a4
    - name: copy template to servers
      template:
        src: httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
      tags: a5
      notify: restart httpd
  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted


## 大型清单管理
> 查询主机db1.example.com是否在清单inventory1内
`ansible db1.example.com -i inventory1 --list-hosts`

>配置并行-f或--forks
```
	ansible web -f 10 -m setup
	ansible-playbook test.yml --forks 10

```
>滚动更新，使用关键字：serial

### 插入剧本或任务：
>插入剧本：`import_playbook`
>插入任务：`import_tasks`


### 角色管理role

>配置role目录，在ansible目录下，创建目录roles，以及相应的目录

`mkdir -pv roles/{web,db}/{defaults,tasks,files,templates,meta,handlers,vars}`

>安装tree，查看文件树型结构


>配置ansible.cfg，加入roles的路径
```
	vim ansible.cfg
		[defaults]

		roles_path=/home/leon/my-ansible/roles

````

### 角色功能的使用：

>案例：
>>按要求使用template模块修改以下主机http端口，并实现访问
* 1.安装http,并修改web01的http端口为8080
* 2.安装http,并修改db01的http端口为888
* 3.在防火墙开启的状态下，能够访问http://web01:8080
* 4.在防火墙开启的状态下，能够访问http://db01:888

>> 制作httpd.conf.j2模板文件
* 1.修改Listen {{ http_port }}
* 2.将httpd.conf.j2 拷贝到 roles/web/templates目录
`cp httpd.conf.j2 roles/web/templates`


>编写相关角色的tasks/main.yml
>> web角色：vim roles/web/tasks/main.yml
```
	- name: start firewalld
	  service:
	    name: firewalld
	    state: started
	    enabled: true
	- name: set 8080 port for firewalld
	  firewalld:
	    ports: 8080/tcp
	    permanent: true
	    immediate: true
	    state: enabled
	- name: install httpd pakeage
	  yum:
	    name: httpd
	    state: present
	- name: copy httpd.conf.j2 to web roles
	  template:
	    src: httpd.conf.j2
	    dest: /etc/httpd/conf/httpd.conf
	  notify: restart httpd
```


>>db角色：vim db/tasks/main.yml
```
	- name: start firewalld
	  service:
	    name: firewalld
	    state: started
	    enabled: true
	- name: set 888 port for firewalld
	  firewalld:
	    port: 888/tcp
	    permanent: true
	    immediate: true
	    state: enabled
	- name: install httpd pakeage
	  yum:
	    name: httpd
	    state: present
	- name: copy httpd.conf.j2 to web roles
	  template:
	    src: httpd.conf.j2
	    dest: /etc/httpd/conf/httpd.conf
	  notify: restart httpd
```

> 编写handlers目录的main.yml文件
```
	vim roles/web/handlers/main.yml
	vim roles/db/handlers/main.yml
	
	
	- name: restart httpd
	  service:
	    name: httpd
	    state: restarted
```

> 编写vars目录变量main.yml文件
```
	vim roles/web/vars/main.yml
	
	http_port: 8080
	
	vim roles/db/vars/main.yml
	
	http_port: 888
```

> 编写roles入口（主）剧本：vim 22.yml
```
	- hosts: 192.168.43.66
	  roles:
	    - web
	
	- hosts: 192.168.43.110
	  roles:
	    - db

```