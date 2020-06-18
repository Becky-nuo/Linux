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





