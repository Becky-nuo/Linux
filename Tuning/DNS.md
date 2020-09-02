# Linux的DNS服务安装与配置

## 实验目的
>了解DNS域名解析服务器原理；
>掌握Linux的DNS服务安装与配置；

## 实验内容
>安装DNS服务；
>配置DNS服务，为虚拟机解析域名；

## 实验要求
>客户端能够解析相应的域名地址；

## 准备知识
### DNS（Domain Name System）域名系统（域名解析服务）
>万维网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的IP数串。通过域名，最终得到该域名对应的IP地址的过程叫做域名解析（或主机名解析端口：TCP 53 /UDP 53；

>功能：每个IP地址都可以有一个主机名，主机名由一个或多个字符串组成，字符串之间用小数点隔开。有了主机名，就不要死记硬背每台IP设备的IP地址，只要记住相对直观有意义的主机名就行；

### DNS空间结构
	
>Root Domain（根域):
>>在DNS域名空间里，根域只有一个，它没有上级域，以圆点“.”来表示；

>Top-level Domain（顶级域）:
>>在根域之下的第一级域便是顶级域，在FQDN（Fully Qualified Domain Name 完全限定域名：同时带有主机名和域名的名称）中，各级域之间都是以原点“.”分隔，如www.baidu.com，“www”为主机名，“.com”就是顶级域,“.com”后面还有个“.”为根域，行业中一般不显示；

>subdomain（子域）：
>>除了根域和顶级域之外的，都为子域。如www.baidu.com，“baidu”就为子域；

### DNS查询模式
>主要有两种模式：迭代查询和递归查询；

>递归查询是一种DNS 服务器的查询模式，在该模式下DNS 服务器接收到客户机请求，必须使用一个准确的查询结果回复客户机；如果DNS 服务器本地没有存储查询DNS 信息，那么该服务器会询问其他服务器，并将返回的查询结果提交给客户机；

>DNS 服务器另外一种查询方式为迭代查询，DNS 服务器会向客户机提供其他能够解析查询请求的DNS 服务器地址，当客户机发送查询请求时，DNS 服务器并不直接回复查询结果，而是告诉客户机另一台DNS 服务器地址，客户机再向这台DNS 服务器提交请求，依次循环直到返回查询的结果；


### 系统自带的域名解析文件
>Linux系统有自带的，优先级最高的域名解析文件，当要解析域名时，系统会优先从该文件中查询结果；
>Linux域名解析文件位置：`/etc/hosts`

>windows的hosts文件路径：`c:\Windows\System32\drivers\etc\hosts`


## 实验步骤
#### 实验拓扑图

### 配置固定IP地址及主机名
>DNS Server
>>vim /etc/sysconfig/network-scripts/ifcfg-ens33 ，IP设为192.168.10.2
>>hostnamectl set-hostname dns		#主机名设为dns

>Linux client
>>vim /etc/sysconfig/network-scripts/ifcfg-ens33 ，IP设为192.168.10.2
>>hostnamectl set-hostname test		#主机名设为test


### 在DNS Server上安装DNS服务
>安装必要软件：`yum install bind -y`

>配置named.conf 
>>打开配置文件：`vim /etc/named.conf`
>>修改成

>打开配置文件：`vim /etc/named.rfc1912.zones`

>/var/named复制文件 `mv  localhost.localhost  jisuanji.localhost`

>复制文件 `mv  localhost.loopback  jisuanji.loopback`

>重启服务，加入开机自启
```
systemctl restart named
systemctl enable named
```

### 在Linux client上设置DNS服务器
>修改resolv.conf，设置DNS Server为DNS服务器；
>>vim /etc/resolv.conf
>>加入以下内容：
```
search xf.com					#搜索域为xf.com
nameserver 192.168.10.1		#dns服务器为192.168.10.1
```

### 客户端测试
```
nslookup
dns.xf.com
test.xf.com
```

>或直接ping测试

>尝试：关闭dns服务，再ping测试
`systemctl stop named`


### 添加反解记录
>在DNS Server上添加反解区域代码
`vim /var/named/dns.db`

>为数据库文件添加反向解析记录
`vim /var/named/dns.db`


>重启服务`systemctl restart named`
>在客户端测试：
```
curl
elinks
nslookup
192.168.10.1
```
