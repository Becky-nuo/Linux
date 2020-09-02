# linux密钥管理
## 实验目的
>了解SSH服务原理，掌握SSH控制规则的配置；

## 实验内容
>安装openssh，配置openssh服务进行相关ssh登录控制；

## 实验要求
>记住SSH监听端口号，openssh配置文件路径，以及了解配置文件各项功能并能根据实际应用进行相关配置；

## 准备知识
### 远程管理：
>在早期，远程管理基本都采用Telnet，但这种明文传输协议非常不安全。因此SSH协议的出现解决了许多在远程管理方式中产生的问题；
### SSH：
>SSH采用了多种认证加密方式，解决了传输中数据加密和身份认证等问题，比较有效的防止网络嗅探与IP欺骗等问题；
>SSH服务默认监听端口号：22；

### openssh：
>Linux/Unux采用openssh程序来实现SSH协议。Server与Client两端都需要安装openssh（即C/S架构）；

## 实验步骤
>安装软件（Centos7已自带安装）
`yum install openssh -y`

### 配置文件解读
>配置文件路径：
>>Server端：`/etc/ssh/sshd_config`
>>Client端：`/etc/ssh/ssh_config`

>配置Server端配置文件：
```vi /etc/ssh/sshd_config

第17行，监听的端口号，默认是22；
第18行，设置SSH服务器绑定的IP地址；

第49号，是否允许管理员登录，默认yes（出于安全性考虑，必要时设置为no）；

第78行，是否允许空密码登录，默认no；
第79行，是否使用密码口令认证方式，默认为yes；
```

### 其他控制：
>指定允许的用户登录,在sshd_config配置文件未行，添加：AllowUsers 用户名；
>指定用户不允许登录,在sshd_config配置文件未行，添加：DenyUsers 用户名；
>指定允许的IP登录:
`vi /etc/hosts.allow sshd:192.168.1.10:allow`

>拒绝所有IP登录:
`vi /etc/hosts.deny	sshd:ALL`

>>其中，3与4两个条件相加，即表示仅允许192.168.1.10登录，其他拒绝；

>配置完成，需要重启服务：`systemctl restart sshd`


### SSH的登录方式
>基于口令登陆
>>例：`ssh 192.168.1.1`
>>或带用户名登录：`ssh root@192.168.1.1`

>基于密钥方式登录
>>在当前用户下生成密钥：ssh-keygen（当提示输入口令时，可以为空）；

>`cd ~/.ssh`

>>其中，id_rsa 为私钥 id_rsa.pub为公钥将公钥复制到远程主机192.168.1.1； 
>>`ssh-copy-id root@192.168.1.1`

>测试：`ssh root@192.168.1.1`

>登录远程主机后，可查看刚复制的密钥与公钥一致：
`less ~/.ssh/authorized_keys`

