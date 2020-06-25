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
>登录zabbix，将本机加入监控；
>选择监控模板，选择监控项；
>添加linux客户端监控；
>登录zabbix，用户Admin，密码zabbix；

>启用本机监控功能,顺序点击“配置”-“主机”-“停用的”，开启本机监控；

>选择监控模板，选择监控项,单击主机，可以看到默认配置项，无须修改；

>查看模板；

>选择监控项，点击“主机”，点击“监控项”；

>勾选需要监控的项目后，点击“启用”；

>返回主页，查看监控效果；

>添加linux客户端监控：

>在需要监控的linux客户端安装zabbix-agent:
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

>确认报警脚本的目录:
```
vim /etc/zabbix/zabbix_server.conf
AlertScriptsPath=/usr/local/zabbix/alertscripts
```

>在报警脚本目录，编写邮件脚本mail.py；

>发送txt文本邮件
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

>zabbix web端——创建媒体类型；
>用户指定媒介；
>创建动作；

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


###实验流程：


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

>测试：
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













