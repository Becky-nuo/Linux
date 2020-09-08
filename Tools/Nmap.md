# nmap

> nmap是一个网络连接端扫描软件，用来扫描网上电脑开放的网络连接端。确定哪些服务运行在哪些连接端，并且推断计算机运行哪个操作系统（这是亦称 fingerprinting）。它是网络管理员必用的软件之一，以及用以评估网络系统安全。

> map支持的四种最基本的扫描方式：
	>> TCP connect()端口扫描（-sT参数）。
	>> TCP同步（SYN）端口扫描（-sS参数）。
	>> UDP端口扫描（-sU参数）。
	>> Ping扫描（-sP参数）。

## 功能  
>* 主机发现,识别网络上的主机,如，列出响应TCP和/或ICMP请求或打开特定端口的主机。
>* 端口扫描,枚举目标主机上的开放端口。
>* 版本检测,询问远程设备上的网络服务以确定应用程序名称和版本号。
>* OS检测,确定网络设备的操作系统和硬件特性。
>* 可与脚本进行脚本交互,使用Nmap脚本引擎（NSE）和Lua编程语言。


## Nmap 安装   
### nmap的纯扫描  

> 默认情况下，nmap会发出一个arp ping扫描（如果交换机是ommited，则为-sP），并且它还会扫描特定目标的打开端口，范围为1-10000

>命令的语法：nmap <目标IP地址>

>其中：目标IP地址=您的目标机器的指定IP


### NMAP普通扫描增加输出冗长（非常详细）

>命令的语法：nmap -vv 10.1.1.254

>其中：-vv =切换以激活日志所需的非常详细的设置


### Nmap的自定义端口扫描

>Nmap的扫描范围从1-10000的端口默认情况下，为了更具体地指定要扫描的端口，我们将使用-p开关来指定我们的端口扫描范围，优点是它会更快扫描少量端口而不是预定义的默认值

>语法：nmap -p（范围）<目标IP>

>其中:-P是端口交换机（范围）是1-65535的端口数 <目标IP>是特定目标IP地址

### Nmap的指定的端口扫描
>明确地指定要扫描的目标端口，如想扫描端口80,443,1000,65534，这些端口适合80-65534这个特定范围，但是如果必须使用该范围内的所有端口都可以使用，就可以再次使用-p开关，这时使用指定端口的一组手指.

>语法：nmap -p（port1，port2，port3等...）<目标IP>

>其中:-P是端口交换机（端口1，端口2，端口3等...）想要扫描的端口列表（必须用逗号分隔） <目标IP>是特定目标IP地址


##NMAP ping
>在nmap中，您也可以使用-sP开关（Arp ping）执行PING命令，这与windows / linux ping命令类似

>语法：nmap -sP <目标IP地址>
>其中：-sP是平扫描开关 <目标IP>是您的特定目标IP地址


### Nmap Traceroute

>跟踪路由用于检测您的计算机数据包从路由器到ISP的路由到互联网直至其特定目的地。

>语法:nmap --traceroute <ip地址>
>其中:--traceroute是traceroute切换命令 <targetIP>是您的特定目标IP地址

### NMAP ping扫描（网络主机dicovery）
>Ping扫描在您正在进入的给定已知网络上完成，以便发现其他计算机/设备那个网络非常相同。

>语法：Nmap -sP <网络地址> </ CIDR>

>其中：-sP是平扫描  


### NMAP操作系统检测

>操作系统检测，以了解何种操作系统当前的目标是非常有用running.Aside从打开的端口，因为信息收集的最重要的数据是concerned.It寻找易受攻击的服务基于端口的特定操作系统目前已开放。

>语法：Nmap -O <目标IP地址>

>其中-O是检测操作系统交换机 <目标IP>是您的特定目标IP地址


### NMAP通用开关

>在一台交换机上完成对端口1-10000的平扫描，还执行操作系统扫描，脚本扫描，路由跟踪以及端口状语从句服务检测。

>语法：Nmap -A <目标IP地址>

>其中-A是大多数功能开关 <目标IP>是您的特定目标IP地址


### NMAP语句
>对特定目标执行nmap的扫描，并将详细程度限制为1-1000的特定端口范围，并执行OS扫描

>语法：nmap -vv -p1-1000 -O <目标ip>

>做一个NMAP扫描扫描特定主机的指定端口：80,23,22和8080，然后执行路由跟踪和OS扫描   
`nmap –p80,8080,22,23 --traceroute -O <目标IP>`


### 安装nmap
```
	cd /opt 
	wget https://nmap.org/dist/nmap-7.70.tgz 
	tar zxvf nmap-7.70.tgz 
	cd nmap-7.70 
	./configure --prefix=/usr/local/nmap 
	yum install gcc-c++ 
	make 
	make install 
```

> 检验安装 
`/usr/local/nmap/bin/nmap -V `


### 使⽤nmap进⾏主机发现 

>ICMP协议: `/usr/local/nmap/bin/nmap -v -n -sn -PE-192.168.1.0/24`

>ARP协议:`/usr/local/nmap/bin/nmap -v -n -sn -PR 192.168.1.0/24 `

### 使⽤nmap进⾏TCP端⼝扫描 
>TCP Connect的⽅式 
`/usr/local/nmap/bin/nmap -v -n -sT --max-retries 1 -p1-65535 192.168.1.108`

>TCP SYB⽅法 
`/usr/local/nmap/bin/nmap -v -n -sS --max-retries 1 -p1-65535 192.168.1.108`

### 使⽤nmap进⾏UDP端⼝扫描 
>`/usr/local/nmap/bin/nmap -v -n -sU -max-retries 1 -p1-65535 192.168.1.108`

### 使⽤nmap识别应⽤ 
/usr/local/nmap/bin/nmap -v -n -sV -p22 192.168.1.108