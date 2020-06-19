#ansible安装与基础命令

```
yum list all *ansible
yum info ansible
```
##配置ssh登录环境

>基于密钥登录
>在inventory文件中指定帐号密码

###ansible 参数：
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

###基于密钥登录
```
cd /etc/ansible
vim hosts
[webserver]
192.168.98.44
```

###设置密钥：
```
ssh-keygen -t rsa
ssh-copy-id -i /root/.ssh/名称.pub root@192.168.98.44
```
>测试免密登录：`ssh 192.168.98.44`


##基础命令

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
##ansible特性：
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


##常用模块：
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

>user： 用户模块；

>>例：
```
添加用户：ansible -m user -a 'name="user1"'；
删除用户：ansible -m user -a 'name="user1" state=absent'；

创建组
ansible all -m group -a 'name=mysql gid=306 system=yes'；
```
>copy：复制模块:
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
>file：文件管理模块
```
path：指定文件路径;
	
创建链接文件：
	state=link：创建链接文件；
	src=指明源文件；
	path：指明链接文件路径；
```
>>例：
```
ansible all -m file -a 'owner=mysql group=msyql mode=644 /tmp/fatab.ansible

ansible all -m file -a 'path=/tmp/fstab.link src=/tmp/fatab.ansible state=link'
```

>ping 网络测试模块

>>例：ansible all -m ping 

>service 服务管理模块:
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

>shell  变量模块（command不支持变量参数）
```

ansible all -m user -a 'name=user1'
尝试：ansible all -m command -a 'echo 123456|passwd --stidn user1'
ansible all -m shell -a 'echo 123456|passwd --stidn user1'
```

>script 脚本模块；

>>注意：基于相对路径方式使用；

>>创建一个本地脚本；
```
vim /tmp/test.sh
#!/bin/bash
echo "hello ansible" > /tmp/script.ansible
useradd user2
chmod +x /tmp/test.sh

cd /tmp
ansible all -m script -a "test.sh"
```
>yum 安装程序包模块；
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




