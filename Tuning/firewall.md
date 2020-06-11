#linux防火墙管理

##实验目的
>了解Linux系统的软件防火墙工作原理，掌握Firewalld的配置方法；

##实验内容
>安装与配置firewalld

##实验要求
 >记住firewalld的九大区域，能够使用firewalld进行相关控制；

##准备知识
>防火墙（Firewall）：计算机网络中的防火墙通常连接两个网络，是外部Internet和内部网络之间的交汇点,同时也是一道屏障；
>主要功能：实施安全策略、过滤传输数据、记录Internet活动、IP地址转换、保护内部网络信息安全；

###类型：
>包过滤防火墙：对通过它的每一个数据包，根据事先制订好的规则，对它的源地址、目的地址以及相应的端口进行判断，把不合规则的数据包都过滤掉 
>>优点：处理速度快，对用户透明

>>缺点：不能实现用户级认证日志，记录不完善，无法防御IP欺骗，对特殊协议不适用

>应用网关防火墙（代理）：核心是代理技术，即代替客户处理对服务器连接请求。
>>优点：安全性高，过滤规则灵活，日志记录详细

>>缺点：处理速度慢，对用户不透明，代理服务类型有限

>状态检测防火墙（动态包过滤防火墙）：
>>在防火墙核心部分建立状态连接表，并将进出网络的数据当成一个个的会话，利用状态表跟踪每个会话状态。
>>首先根据规则表来判断是否允许这个数据包通过，如果不符合规则就立即丢弃；如果符合，再将当前数据包和状态信息，与前一时刻的相比较，然后根据比较结果判断数据包是否能够通过;

###防火墙结构：
>屏蔽路由器（包过滤）：
![image](https://github.com/Becky-nuo/git-test/blob/master/images/firewall/001.png)


>双穴主机（双宿网关）：
![image](https://github.com/Becky-nuo/git-test/blob/master/images/firewall/003.png)

>屏蔽主机防火墙：
![image](https://github.com/Becky-nuo/git-test/blob/master/images/firewall/004.png)


>其它结构：
![image](https://github.com/Becky-nuo/git-test/blob/master/images/firewall/005.png)


>软件防火墙：Linux防火墙采用包过滤技术，大致经历了四个发展阶段：
>静态防火墙：需要自行制订规则，每次改变后都需要重启防火墙；
```
ipfwadm	
ipchains	
iptables
```
>动态防火墙：在更改规则后，无需重启: firewalld（RHEL 7新纳入的防火墙）

>firewalld的九大区域：
```
1.public：默认区域（公共区域），只有设置为允许的通信才可以通过
2.trusted：信任区域，允许所有通信通过
3.drop：丢弃区域，拒绝所有通信
4.block：拒绝所有外部通信，允许内部通信
5.external：nat区域，开启nat和端口映射
6.dmz：非军事工域，允许外部访问的服务器
7.work：工作区域
8.home：家庭区域
9.internal：内部区域
```

##实验步骤

###实验拓扑图
![](http://images/firewall/005.png)
	
>Centos 7 里，firewalld是默认启动的（而iptables是默认关闭，两者不能同时开启）

###查看规则
>查看firewalld状态 ：`systemctl status firewalld`

>查看当前默认区域：`firewall-cmd --get-default-zone`

>查看默认区域的所有规则：`firewall-cmd --list-all`

>查看所有区域：`firewall-cmd --get-zones`

>查看接口所在区域：`firewall-cmd --get-zone-of-interface=eth0`

>指定区域查看该区域的所有规则：`firewall-cmd --zone=trusted --list-all`

>查看所有区域的规则：`firewall-cmd --list-all-zones`

###修改规则
>修改默认区域：`firewall-cmd --set-default-zone=trusted`

>在默认区域添加服务：`firewall-cmd --add-service=http`

>将网卡添加到指定区域：`firewall-cmd --zone=trusted --add-interface=eth0`

>修改网卡所在区域：`firewall-cmd --zone=public --change-interface=eth0`

>在默认区域指定端口和协议：`firewall-cmd --add-port=80/tcp`（建议在public区域设置）


###移除规则
>指定区域移除http：`firewall-cmd --zone=trusted --remove-service=http`

>将网卡移除出指定区域：`firewall-cmd --zone=public --remove-interface=eth0`

###模拟场景一：http服务访问控制
>在R上开启防火墙，验证Client B能否通过R访问ClientA 的http服务

>Client A：
>>安装http服务：`yum install httpd -y`

>>配置http服务：
```
echo “hello” > /var/www/html/index.html  
systemctl start httpd
```

>R 防火墙：
>>查看当前默认区域规则：`firewall-cmd --list-all`

>可以看到，当前默认区域public没有允许http服务通过，所以Client B是无法访问A的http服务的；

>方法一：将http服务加入默认区域：`firewall-cmd --add-service=http`
>方法二：将默认区域修改为trusted区域（信任）：`firewall-cmd --set-default-zone=trusted`

###模拟场景二：ICMP协议控制（ping）
>默认情况下，firewalld允许所有的ICMP类型（ping）协议通过，也可以做控制:
`firewall-cmd --add-icmp-block=echo-request`

>维护注意事项：
>>所有修改过的规则，在重启服务后将失效，要永久生效，需要用参数“--permanent”，然后重启服务；比如：
```
firewall-cmd --permanent --add-service=http
systemctl restart firewalld
```
