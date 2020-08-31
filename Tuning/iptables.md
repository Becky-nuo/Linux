# iptables实验
>开源防火墙。

>本实验需要两台服务器：一台服务机(172.23.65.130)，一台客户机(192.168.1.106)

## 实验一、禁止客户机ping服务机
>step1：先确认当前两台主机可以互相ping通
>step2：设置INPUT链的规则，丢弃ping协议包 
`sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP`

>step3：查看设置的规则 
`sudo iptables -n -L`

>step4：两台主机互ping

>step5：删除丢弃ping包的规则，并查看规则 
`sudo iptables -D INPUT -p icmp --icmp-type echo-request -j DROP`

>step6：设置INPUT链的规则，允许ping协议包通过防火墙 
`sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT`

>重复step3、step4查看实际效果。


## 实验二、限制客户机使用ssh（22端口）登录服务机
>step1：在客户机使用ssh工具（putty/xshell）登录服务器

>step2：设置限制规则 
`sudo iptables -A INPUT -p tcp -s 192.168.1.106 --dport 22 -j DROP`

>step3：查看设置的规则 
`sudo iptables -n -L`

>step4：再次在客户机使用ssh工具（putty/xshell）登录服务器
>step5：删除设置的规则 
`sudo iptables -D INPUT -p tcp -s 192.168.1.106 --dport 22 -j DROP`

>step6：设置允许登录的规则 
`sudo iptables -A INPUT -p tcp -s 192.168.1.106 --dport 22 -j ACCEPT`

>step7：尝试在客户机使用ssh工具（putty/xshell）登录服务器


## 实验三、禁止服务机数据输入/输出
>step1：先确认当前两台主机可以互相ping通

>step2：禁止服务机数据输入  
`sudo iptables -A INPUT -j DROP`

>step3：查看设置的规则  
`sudo iptables -n -L`

>step4：客户机服务机互ping

>step5：解除限制  
`sudo iptables -D INPUT -j DROP`

>step6：禁止服务机数据输出  
`sudo iptables -A OUTPUT -j DROP`

>>重复step3

>step7：服务机ping客户机


## 实验四、内网网络地址转换
>有服务器A设置双网卡(10.128.70.112)（10.128.70.114）、服务器B(10.128.70.111)、服务器C(10.128.70.113)且服务器B不连接外网

>step1：设置服务器B的默认网关为服务器A的网卡1地址（10.128.70.112） 
```
	vi /etc/sysconfig/network-scripts/ifcfg-ens33
	GATEWAY=10.128.70.112
	DNS1=10.128.70.112
```

>step2：设置服务器C的默认网关为服务器A的网卡2地址（10.128.70.114） 
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
GATEWAY=10.128.70.114
DNS1=10.128.70.114
```
>step2：在服务器A上启用路由功能。  
`sudo sysctl -w net.ipv4.ip_forward=1`

>step3：在iptables设置转发规则
```
sudo iptables -t filter -A FORWARD -j ACCPECT
sudo iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to 10.128.70.114
```
>>这里eth0是服务器A的网卡2，10.128.70.114是服务器A的网卡2地址此时使用服务器B给C发送数据，在服务器C上看见的地址是服务器A网卡2的地址。

## 实验五、外网网络地址转换
>外网地址转换用于外部用户直接访问无外网IP的服务器( Server B)提供的服务时。例如,外部用户C希望通过互联网访问到 Server B上的 Oracle数据库(监听端口是TCP1521)时,可以使用如下的命令在 Server A上进行目的地址转换设置:

>step1：设置路由规则，将目标为10.128.70.114目标端口为1521的数据包转发给10.128.70.111:1521   
`iptables -t nat -A PREROUTING -d 10.128.70.114 -p tcp -m tcp --dport 1521 -j`

>step2：设置路由规则，将目标为10.128.70.111目标端口为1521的数据包有网卡1进行发送  
`iptables -t nat -A PREROUTING -d 10.128.70.114 -p tcp -m tcp --dport 1521 -j`