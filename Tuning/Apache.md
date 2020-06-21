#Linux的Apache服务

##实验目的
>了解Apache（Httpd）服务的功能；
>掌握Linux的Apache服务安装与配置；

##实验内容
>安装Apache服务；
>配置Apache服务，搭建一个web站点；

##实验要求
>客户端能够访问web站点；

##准备知识
###Apache
>Apache软件基金会（也就是Apache Software Foundation，简称为ASF）是专门为运作一个开源软件项目的>Apache 的团体提供支持的非盈利性组织，这个开源软件的项目就是 Apache 项目；

>Apache HTTP服务器项目主要致力于为现代操作系统开发和维护开源的HTTP服务器，其中包括Unix和Windows NT；
>>这个项目的主要目标是提供一个可以与当前的HTTP标准同步提供安全、高效和可扩展的服务器的HTTP服务；

###Apache HTTP的三种工作模式
####Prefork MPM
>默认模式，服务器为每个用户的连接开启一个进程做响应，对系统资源占有量比较大；
>适用于：并发连接数较小，页面程序较多的站点；

####Worker MPM
>由apache主进程开启多个子进程，每个进程内开多个线程，每个线程响应一个客户连接；
>适用于：并发连接数多，点击量较大的站点；

####Event MPM
>由apache主进程响应客户的连接，当客户登录时，再针对每个帐号开启一个进程，进程内多线程来完成；

###HTTP
>Http是Apache超文本传输协议(HTTP)服务器的主程序；被设计为一个独立运行的后台进程，它会建立一个处理请求的子进程或线程的池；
>功能：WEB浏览、下载；
>主配置文件：`/etc/httpd/conf/httpd.conf`；
>端口：80；



##实验步骤
###实验拓扑图

###配置固定IP及主机名，配置本机域名解析

####Web Server
```
vim /etc/sysconfig/network-scripts/ifcfg-ens33 ，IP设为；
192.168.10.1
hostnamectl set-hostname web		#主机名设为web；
```

####Linux client
```
vim /etc/sysconfig/network-scripts/ifcfg-ens33 ，IP设为；
192.168.10.2
hostnamectl set-hostname test		#主机名设为test；
```

####由于没有DNS服务器，这里使用本机的hosts文件作域名解析
>修改本机hosts文件，加入web.xf.com的解析记录；
```
vim /etc/hosts
192.168.10.1 web.xf.com

ping 测试
ping web.xf.com
```


###安装软件，配置第一个web页面
>安装软件:`yum install httpd -y`
>进入主页目录，编辑主页文件：/var/www/html ；
```
cd /var/www/html
vim index.html
随便输入内容，比如“这是我人生搭建的第一个网页”，保存退出；
```

>启动服务：`systemctl start httpd`
>安装linux浏览器软件:`yum install elinks -y`
>测试:`elinks http://web.xf.com`
>测试成功，按q，退出elinks界面；


###修改apache工作模式
>vim /etc/httpd/conf.modules.d/00-mpm.conf,根据需要取消相对工作模式前的“#”号注释即可；

>查看当前apache工作模式：`httpd -V | grep "Server MPM'`
>打开配置文件：vim /etc/named.conf
>修改成最简配置；


###添加正解区域：

>创建dns数据库文件dns.db:`cd /var/named`
>复制系统自带模板:`cp named.localhost dns.db`
>编辑dns.db，加入解析记录:`vim /var/named/dns.db`

>修改数据库文件的属主，属组:
```
cd /var/named
chown named.named dns.db
```

>重启服务，加入开机自启:
```
systemctl restart named
systemctl enable named
```

###在Linux client上设置DNS服务器
>修改resolv.conf，设置DNS Server为DNS服务器:
```
vim /etc/resolv.conf

加入以下内容：
	search xf.com					#搜索域为xf.com
	nameserver 192.168.10.1		#dns服务器为192.168.10.1
````

###客户端测试
```
nslookup
dns.xf.com
test.xf.com
```

>或直接ping测试

>>尝试：关闭dns服务，再ping测试:`systemctl stop named`


###添加反解记录
>在DNS Server上添加反解区域代码:`vim /var/named/dns.db`

>为数据库文件添加反向解析记录:`vim /var/named/dns.db`

>重启服务:`systemctl restart named`

>在客户端测试：
```
nslookup
192.168.10.1
```