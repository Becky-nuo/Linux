#Zabbix认证配置

##zabbix 概述

###监控系统：

>SNMP协议（Simple Network Manager Protocol）；

>简单网络管理协议（SNMP） 是专门设计用于在 IP 网络管理网络节点（服务器、工作站、路由器、交换机及HUBS等）的一种标准协议，它是一种应用层协议；



>一个SNMP管理的网络由下列三个关键组件组成：
>>网络管理系统（NMS，Network-management systems）；
>>被管理的设备（managed device）；
>>代理者（agent）；


>SNMP是管理进程（NMS）和代理进程（Agent）之间的通信协议。它规定了在网络环境中对设备进行监视和管理的标准化管理框架、通信的公共语言、相应的安全和访问控制机制；
>>网络管理员使用SNMP功能可以查询设备信息、修改设备的参数值、监控设备状态、自动发现网络故障、生成报告等；


>基于snmp协议实现的监控工具：
>>cacti（无agent）--偏向与数据采集展示工具，采集数据、保存数据、数据展示、数据分析及报警；

>>nagios       --偏向与报警工具，根据阀值判断状态正常与否，zabbix等；



>>其他监控方式：ssh等，如ansible
>>zabbix-server默认监听端口：10051
>>ss -tnpl
>>zabbix-agent默认监听端口：10050

>配置agent

>>修改配置文件zabbix_agentd.conf，修改zabbix server的ip地址
`vim /etc/zabbix/zabbix_agentd.conf`
	
>>Server=			#zabbix服务端地址，根据实际情况修改



>>加入开机自启并启动zabbix客户端
```
systemctl enable zabbix-agent
systemctl start zabbix-agent
```

###zabbix监控范围

>设备/软件:服务器、路由器、交换机、输入/输出设备等；

>事故：数据库当机、服务不可达、备份停止等；

>敏感事件：磁盘满、内存不足、cpu利用率等；


###zabbix架构

>web Gui：网页接口（界面）；
>DB：数据库；
>Server：服务；
>监控手段：web、协议（如snmp）、Agent（客户端）；

----------------------------------------------

###zabbix术语

>host：主机，要监控的网络设备；
>host group：主机组；
>item：监控项，一个特定监控指标的相关数据，这些数据来自于监控对象，对于item是zabbix进行数据收集的核心，没有item就没有数据。每个item都由key进行标识；

>trigger：触发器，一个表达式，用于评估某监控对象的某特定item内所接收到的数据是否在合理范围内（阀值），由此阀值判断状态，ok或problem；
>event：事件，即发生的一个值得关注的事情，例如触发器的状态转变，新的agent或重新上线的agent的自动注册等；
>action：动作，对于特定事件先定义的处理方法，通过包含操作（如发送通知）和条件（何时执行操作）；

>escalation：报警升级，发送报警或执行远程命令的自定义方案，如每隔5分钟发送一次警报，共发多少次等；
>media：发送通知的手段或通道，如email等；

>notification：通知。通过选定的媒介向用户发送的有关某事件的信息；
>remote command：预定义命令，可在被监控主机处于某特定条件下时自动执行；
>template：模板，用于快速定义被监控主机的预设条目集合，通常包含了item，trigger，graph，screen，application以及low-level discovery rule等；
>>模板可以直接链接至单个主机；

>application：应用程序，一组item的集合；
>web scennario：web场景，用于检测web站点可用性的一个或多个http请求；
>frontend：前端，zabbix的web接口（界面）；



###监控软件应该具备的基本功能：

>zabbix:数据采集-->数据存储-->数据展示/分析-->报警；


>数据采集：
```
SNMP
agent
ICMP/SSH/IPMI
```
>数据存储：mysql/pgsql/oracle/mariadb ；

>数据展示：java/php/移动app

>报警：
```
mail（smtp）/短信
wechat/qq/msn
```

###触发器（Triggers）

>监控项（item）：仅负责收集数据，而通常收集数据的目的还包括在某指标对应的数据超出合理范围时给相关人员发送告警信息；



>一个触发器同一个表达式构成，它定义了监控项所采取的数据的一个阀值；

>每一个触发器仅能关联一个监控项，但一个监控项同时使用多个触发器；


>工作过程：一旦某次采集的数据超出了触发器定义的阀值，触发器状态将会转换为“problem”;而当采取的数据再次回归合理范围时，则转回为“ok”；


>触发器的基本表达式：{<server>:<key>.<function>（<parameter>）}<operator><constant>；


>>说明：
```
server：主机；
key：主机上关系的相应监控项的key；
function：评估采集到的数据是否在合理范围内时所使用的函数，其评估过程可以根据采取的数据、当前时间及其它因素进行；

目前支持的函数有：
avg、count、change、date、dayofweek、delta、diff ....等等；

parameter：函数参数；

大多数数值函数可以接受秒为其参数，如果在数值参数前使用#号做为前缀，则表示为最近几次的取值；

比如：
sum（300），表示300秒内所有取值之和；
sum（#10），表示最近10次取值之和；

此外，函数还支持使用第二参数，比如max（1h，7d），表示将返回一周之前的最大值；
```

>触发器例子：

>>{web-server:system.cpu.load[all,avg1].last(0)}>3；

>>web-server主机上所有CPU的过去1分钟内的平均负载的最后一次取值大于3时将触发状态变换；

>>注：对于last函数，last（0）相当于last（#1）；


###实验：

>创建一个触发器：对网卡入方向的流量最后一次采集数据时的阀值大于1K时，触发（警告）事件；

>配置->主机->触发器->创建触发器；

```
function：
Last（most recent）T value is > N

N=1024
```

>创建触发器可用的各属性说明
```
Name：触发器名称，可以使用宏，如$1、$2....$9等；
Expression：逻辑表达式，用于评估触发器状态；
Multiple PROBLEM events generation:依赖于当前触发器的Problem状态生成其他事件；
Description：描述信息；
URL：在聚合图形的“Status of Trigger”中显示的内容链接；
Severity：严重级别；
Enable：启用与否；
```


###zabiix通知（报警）功能

>一般需要两个步骤：
>>1.定义所需要的“媒介（media）”，如邮件、SMS等；
>>2.配置一个“动作（action）”（措施），发送至媒介；

>>动作（action）又由“条件”和“操作”组成，当满足条件时，执行操作一般体现为“发送通知”，“执行远程命令”；



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

>下载安装包

`wget -r -np -nH -R index.html http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/`

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
wget 
http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm
rpm -ivh zabbix-release-3.2-1.el7.noarch.rpm
yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent
```

>导入数据表结构:
```
cd /usr/share/doc/zabbix-server-mysql-3.2.11/
zcat create.sql.gz | mysql -uzabbix -p123456 zabbix
```
>修改zabbix_server.conf配置文件:
```
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

>访问地址：`http://server-ip/zabbix/setup.php`
>>注：server-ip为本机ip地址

>注：下面的所有配置必须都为ok才可以；


>>配置zabbix的数据库相关信息，此处的密码为“123456”；
>>确认所有设置信息；


>>登录zabbix,默认用户为admin，密码为zabbix；

>>设置字体显示为中文，点击界面右上角的用户；


##Zabbix磁盘监控
###实验目的

>掌握zabbix添加监控主机的方法；

###实验内容
>添加主机磁盘监控；
>添加linux客户端监控；

###实验要求
>监控本机；
>监控linux客户端；

###实验步骤

>登录zabbix，用户Admin，密码zabbix；

>>启用本机监控功能,顺序点击“配置”-“主机”-“停用的”，开启本机监控；


>选择监控模板，选择监控项,单击主机，可以看到默认配置项，无须修改；


>查看模板；

>选择监控项；
>>点击“主机”，点击“监控项”；

>>勾选需要监控的项目后，点击“启用”；

>>返回主页，查看监控效果；


>添加linux客户端监控：

>>在需要监控的linux客户端安装zabbix-agent:
`yum install -y zabbix-agent`


>修改配置文件zabbix_agentd.conf，修改zabbix server的ip地址:
```
vim /etc/zabbix/zabbix_agentd.conf

Server=192.168.80.150			#zabbix服务端地址，根据实际情况修改
ServerActive=192.168.80.150	 #zabbix服务端地址，根据实际情况修改
```	

>加入开机自启并启动zabbix客户端:
```
systemctl enable zabbix-agent
systemctl start zabbix-agent
```

>添加客户机的监控,界面操作：配置--主创—创建主机；
	
>填写客户机信息,单击主机，选择模板“Template OS Linux”；

>选择监控项；


##Zabbix邮件告警
###实验目的
>掌握Zabbix邮件告警相关设置；

###实验内容
>下载sendEmail
>编写mail.sh邮件脚本；
>配置邮件服务；


###实验要求
>监控本机；
>监控linux客户端；

###实验步骤
>下载sendEmail
`wget http://caspian.dotconf.net/menu/Software/SendEmail/sendEmail-v1.56.tar.gz`

>>解压软件，然后将sendemail复制到/usr/local/bin/目录下，并加上可执行权限，然后修改用户和群组：
```
tar -xvf sendEmail-v1.56.tar.gz
cp sendEmail-v1.56/sendEmail /usr/local/bin/
chmod 755 /usr/local/bin/sendEmail
chown zabbix.zabbix sendEmail
ll /usr/local/bin
```

>>确认报警脚本的目录:
```
vim /etc/zabbix/zabbix_server.conf
AlertScriptsPath=/usr/local/zabbix/alertscripts
```

>>在报警脚本目录，编写邮件脚本mail.py；

>>发送txt文本邮件
```
import smtplib
from email.mime.text import MIMEText
from sys import argv

mailto_list=[]
mail_host="smtp.qq.com:25"  #设置服务器
mail_user="XX@qq.com"     #发件用户名(换成自己的)
mail_pass="XXXXX"   #口令（换成自己的） 
mail_postfix="qq.com"  #发件箱的后缀
debug_level=0       #是否开启debug

def send_mail(to_list,sub,content):
    me=mail_user
    msg = MIMEText(content,_subtype='plain',_charset='utf-8')
    msg['Subject'] = sub
    msg['From'] = me
    msg['To'] = ";".join(to_list)
    try:
        server = smtplib.SMTP()
        server.set_debuglevel(debug_level)
        server.connect(mail_host)
        server.login(mail_user,mail_pass)
        server.sendmail(me, to_list, msg.as_string())
        server.close()
        return True
    except Exception, e:
        print str(e)
        return False
if __name__ == '__main__':
    try:
        mailto_list=argv[1].split(';')
        sub=argv[2]
        content=argv[3]
    except:
        print "python send_mail.py 'XXXXX@qq.com' sub content"
        exit()

    if send_mail(mailto_list,sub,content):
        print "发送成功"
    else:
        print "发送失败"

```

>zabbix web端——创建媒体类型，管理-创建；
>用户指定媒介；
>创建动作，配置-动作-创建；

>测试：
>>手动关闭监控中的某台主机，查看报警情况；


##Zabbix服务监控

##实验目的
>掌握zabbix添加nginx服务的方法与配置

###实验内容
>为客户机安装nginx服务
>zabbix Server添加客户机的nginx服务监控

###实验要求
>客户机成功安装nginx服务并能通过浏览器访问
>Zabbix成功监控nginx服务

###实验步骤
>使用zabbix监控nginx服务
![](https://github.com/Becky-nuo/git-test/tree/master/images/Zabbix/001.png)

###实验流程：
![](https://github.com/Becky-nuo/git-test/tree/master/images/Zabbix/002.png)

>在Nginx-Server安装nginx服务
>>下载安装nginx
```
wget http://dl.Fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum install nginx -y
```

>>启动nginx
```
systemctl start nginx
systemctl enable nginx
```

>>测试：
```
systemctl stop firewalld.service 		#停止firewall
systemctl disable firewalld.service	 	#禁止firewall开机启动
```
>>关闭selinux
>>浏览器打开：http://服务器地址


>在zabbix Server添加nginx服务监控

>>点击“Web监测”-“创建Web场景”

>>新增“步骤”

>>点击“添加”

>>新增触发器：“配置”-“主机”“触发器”

>>“创建触发器”

>>查看监测状态：“监测中”-“Web监测”


>模拟nginx服务故障，

>>修改nginx Server上index.html文件后缀，模拟故障
```
cd /usr/share/nginx/html
mv index.html index.html.bak
```

>>打开nginx页面：`http://192.168.80.99`

>>查看zabbix监控：


##监控Windowns

>下载zabbix_agents_3.0.4.win.zip


>修改三个参数：
>>找到conf下的配置文件 zabbix_agentd.win.conf ，修改LogFile、Server、、ServerActive、Hostname这四个参数。具体配置如下：

```
LogFile=c:\zabbix_agentd.log  #默认参数，启动后会自动生成。
Server=192.168.30.141         #被监控主机的IP地址
Hostname=WIN-194215QI0VR      #被监控主机名称
ServerActive=192.168.30.141 
```
>>zabbix server地址 #其中logfile是zabbix日志存放地址。Server是zabbix服务端ip地址。Hostname是本机机器名。

>查看windows主机名称：
>>C:\Users\Administrator>hostname
>>WIN-194215QI0VR


>zabbix-agent语法
```
 -c ：指定配置文件所有位置
 -i ：安装客户端
 -s ：启动客户端
 -x ：停止客户端
 -d ：卸载客户端
```

>启动zabbix-agent服务

>>开始--->运行：cmd （记得以管理员身份运行，否则会造成服务无法成功启动）
```
C:\zabbix\bin\win64>zabbix_agentd.exe -c C:\zabbix\conf\zabbix_agentd.win.conf -i
zabbix_agentd.exe [304]: service [Zabbix Agent] installed successfully
zabbix_agentd.exe [304]: event source [Zabbix Agent] installed successfully
```

>重启服务的方法

>>C:\Windows\system32>net stop "Zabbix Agent"
>>Zabbix Agent 服务已成功停止。


>>C:\Windows\system32>net start "Zabbix Agent"
>>Zabbix Agent 服务正在启动 .
>>Zabbix Agent 服务已经启动成功。



>卸载zabbix-agent （如果需要）
```
C:\zabbix\bin\win64>zabbix_agentd -d
zabbix_agentd [6040]: service [Zabbix Agent] uninstalled successfully
zabbix_agentd [6040]: event source [Zabbix Agent] uninstalled successfully

```

>把服务设置为开机自动启动

>查看zabbix-agnet端口：10050

`C:\Users\Administrator>netstat -an | find "10050"`


>添加监控windows主机


>通过以下方法找到网卡真正名称：
`typeperf -qx | find "Network Interface" | find "Bytes" > c:\network.txt`

>>特别注意：通过适配器看到的网卡名称，和通过命令获取到的网卡名称，有些不一样的地方，要以命令获取到的名称为准，不然有些特殊符号是无法识别的,例如：(R)，PRO/1000，在命令获取到的名称中则是[R]，PRO_1000；


>修改客户端配置文件zabbix_agentd.win.conf:
```
Server=服务端IP
Hostname=当前客户端主机名（可直接填写的本机IP）

并在最下面添加以下内容：

PerfCounter = Net_Incoming,"\Network Interface(Intel[R] PRO_1000 MT Desktop Adapter)\Bytes Received/sec",30
PerfCounter = Net_Outgoing,"\Network Interface(Intel[R] PRO_1000 MT Desktop Adapter)\Bytes Sent/sec",30
```


>重启zabbix客户端服务,在服务器管理器-配置-服务中找到Zabbix Agent重新启动服务或命令:
```
zabbix_agentd.exe –x
zabbix_agentd.exe –s
```

>服务端验证
>>在已经搭好的linux服务端上运行下面命令，正常会返回一个数值:
`zabbix_get -s IP -k "Net_Incoming"  （需要安装zabbix-get）`
















