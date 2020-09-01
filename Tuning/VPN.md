# VPN 实验

## 点到点的虚拟专⽤⽹络 
>点到点的虚拟专⽤⽹络就是把两台处于因特⽹上的服务器使⽤虚拟专⽤⽹络连接起来，⽐如说远程数据库备份 

>步骤⼀：准备第⼀台服务器安装OpenVPN 
```
yum install -y lzo lzo-devel openssl openssl-devel pam pam-devel  

wget http://dl.fedoraproject.org/pub/epel/epel-releaselatest-7.noarch.rpm 

rpm -ivh epel-release-latest-7.noarch.rpm 
yum install -y pkcs11-helper pkcs11-helper-devel 
yum install openvpn
```
>步骤二：复制第⼀台服务器，注意重新⽣成MAC地址

>步骤三：确认每台服务器的IP地址，服务器A（192.168.1.111），服务器B（192.168.1.114）

>步骤四：在服务器A上⽣成静态密码

>步骤五：将key复制到服务器B

>步骤六：创建隧道 

>>服务器A

>>服务器B

>步骤七：验证通道
>>服务器B

>>服务器A


## 创建远程访问的虚拟专⽤⽹络 
>实验背景： 因为疫情原因，公司员⼯需要在家进⾏远程办公，他们需要访问公司内⽹的服务器资源或者办公⽹络资 源。

>实验环境： ⼀台Linux服务器A（双⽹卡，连接外⽹和内⽹） ⼀台Linux服务器B（内⽹） ⼀台windows电脑（外⽹） 

>实验步骤： 

>>步骤⼀：下载证书⽣成⼯具，在Linux服务器上⽣成CA证书、服务器证书、客户端 证书

>>下载⽣成⼯具 https://github.com/OpenVPN/easy-rsa.git ⽣成CA证书 

>>⽣成服务器证书

>>⽣成客户端证书


>>步骤⼆：创建⽇志⽂件夹，同时修改为nobody的权限。

>>步骤三：⽣成ta密钥和dh密钥，同时将服务器证书和ca证书拷⻉到⽬录下 

>>步骤四：创建server.conf⽂件

>>步骤五：创建ccd⽂件夹，在ccd⽂件夹中创建客户端配置⽂件 

>>步骤六：启动服务器 

>>步骤七：检测是否启动成功

>>步骤⼋：安装windows客户端 

>>步骤九：编辑配置⽂件vpnclient.ovpn，以及安装证书

>>步骤⼗：连接VPN ⼩电视变绿表示连接成功 

>>步骤11：设置地址转换，使外⽹windows计算机可以连接到内⽹服务器B,同iptables实验内容  

>>步骤12：进⾏外⽹计算机和内⽹服务器B的连接测试，同VPN实验⼀ 