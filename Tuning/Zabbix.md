#Zabbix认证配置

##安装

###实验目的
>掌握Zabbix的部署过程；
>掌握Zabiix的相关配置；

###实验内容
>基础环境安装；
>maria数据库安装；
>zabbix服务端安装；

###实验要求
>使用Zabbix监控本机磁盘与相关服务；


###实验步骤
>安装maria数据库；
>zabbix数据库相关配置；
>安装zabbix服务端及网页界面设置；


>安装maria数据库:`yum -y install mariadb-server`

>启动与初始化数据库:
```
systemctl start mariadb
systemctl enable mariadb
mysql_secure_installation
```

>>新装数据库无密码，直接回车，继续对话；

>>是否设置数据库管理员登录密码，选择y，密码本文简单设置为：1；

>>是否移除匿名用户，选择y；

>>禁用管理员远程登录，选择n；

>>是否移除测试用的数据库，选择y；
	
>>重新加载权限表，选择y；



>zabbix数据库相关设置
>>创建zabbix数据库、用户并设置数据库登录密码:
```
mysql -uroot -p
create database zabbix default character set utf8;
grant all privileges on zabbix.* to zabbix@localhost identified by '123456';
exit		#退出
```

>安装zabbix服务端及网页界面设置,获取并安装zabbix的yum源：
```
wget http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm
rpm -ivh zabbix-release-3.2-1.el7.noarch.rpm
yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent
```

>导入数据表结构:
```
cd /usr/share/doc/zabbix-server-mysql-3.2.11/
zcat create.sql.gz | mysql -uzabbix -p123456 zabbix
修改zabbix_server.conf配置文件
vim /etc/zabbix/zabbix_server.conf
DBPassword=123456
```

>定义时区:
```
vim /etc/httpd/conf.d/zabbix.conf
php_value date.timezone Asia/Shanghai
```

>设置开机自启动，启动zabbix服务器端:
```
systemctl enable zabbix-server
systemctl restart zabbix-server
```

>设置开机自启动，启动apache:
```
systemctl enable httpd
systemctl start httpd
```

>关闭firewalld，关闭selinux，web界面安装zabbix:
```
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
```

>访问地址：http://server-ip/zabbix/setup.php
>>注：server-ip为本机ip地址

>注：下面的所有配置必须都为ok才可以；


>配置zabbix的数据库相关信息，此处的密码为“123456”；
>>确认所有设置信息；


>登录zabbix,默认用户为admin，密码为zabbix；

>设置字体显示为中文，点击界面右上角的用户；






