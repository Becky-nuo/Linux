#KVM虚拟化技术

##环境部署

###KVM虚拟机网络模式：
>Bridge（网桥）：网桥的基本原理就是创建一个桥接接口br0，在物理网卡和虚拟网络接口之间传递数据，如下图：
![](https://github.com/Becky-nuo/git-test/blob/master/images/KVM/001.png)

>NAT：虚拟接口和物理接口之间没有连接关系，所以虚拟机只能在通过虚拟的网络访问外部世界，无法从网络上定位和访问虚拟机； 
![](https://github.com/Becky-nuo/git-test/blob/master/images/KVM/002.png)

>“透传”：嵌套式虚拟nested是一个可通过内核参数来启用的功能，能够使一台虚拟机具有物理机CPU特性,支持vmx或者svm(AMD)硬件虚拟化；

###实验步骤
>KVM安装前准备
>>安装Centos7.3图形化界面系统，配置本地yum 源检查主机处理器是否支持虚拟化：
`egrep -o 'vmx | svm' /proc/cpuinfo | wc -l`

>>如果返回显示数值是 0，则表示该 CPU 不支持虚拟化;

>为了方便实验，关闭firewall和selinux:
```
systemctl stop firewalld.service
systemctl disable firewalld.service

setenforce 0			#临时关闭selinux

或vim /etc/sysconfi/selinux，将第7行参数“enforcing”改为“disabled”，然后重启服务器，永久关闭selinux
```


>安装KVM	：
```
yum upgrade
yum install qemu-kvm* virt-* libvirt* -y

```
>启动服务并加入开机自启:
```
	systemctl start libvirtd
	systemctl enable libvirtd
```
>检查kvm模块挂载情况：`lsmod |grep kvm_intel`


>开启主机“透传”功能，需要重启服务器:
`echo "options kvm_intel nested=1" >> /etc/modprobe.d/kvm-nested.conf`

>查询透传是否开启（Y或N）:
`cat /sys/module/kvm_intel/parameters/nested`


###创建网桥br0
	
>查看主机网上相关信息：`ip a`	
	

>有4块网块，分别为：
```
enp2s0f0	
enp2s0f1	
enp2s0f2	
enp2s0f3
virbr0：kvm安装完后自带的网桥，主要供NAT网络使用
virbr0-nic：virbr0网桥上的一个子接口，相当与nat网络里的网关接口
```

>使用enp2s0f0做为网桥:
```
	service NetworkManager stop		#关闭图形化自带的NM网络管理工具
	chkconfig NetworkManager off	#关闭开机自启
	
	cd /etc/sysconfig/network-scripts/	#进入网卡配置文件目录
	cp ifcfg-enp2s0f0 ifcfg-enp2s0f0.bak	#备份网卡1配置文件
	cp ifcfg-enp2s0f0 ifcfg-br0		#创建br0网桥配置文件
	vim ifcfg-enp2s0f0				#修改网卡1配置文件
	
TYPE=Ethernet					#设备类型
	BOOTPROTO=none					#不指定ip获取类型（dhcp/static）
	DEVICE=enp2s0f0					#设备名称
	ONBBOT=yes						#开机启动
	BRIDGE=br0						#指定网桥为br0
	wq保存退出

vim ifcfg-br0					#修改网桥bro配置文件

	TYPE=Bridge						#设定设备类型为网桥
	BOOTPROTO=static				#指定IP方式为静态IP地址
	NAME=br0						#名称
	DEVICE=br0						#设备名称
	ONBOOT=yes						#开机启动
	IPADDR=192.168.80.40			#IP地址
	NETMASK=255.255.255.0			#子网掩码 
	GATEWAY=192.168.80.1			#网关
	DNS1=114.114.114.114			#第一个DNS服务器
	DNS2=8.8.8.8					#第二个DNS服务器
```

>重启网络：`systemctl restart network`
>重启libvirt服务：`systemctl restart libvirtd`
>查看网桥：`brctl show`
	

>开启网桥stp树协议，防止环路：`brctl stp br0 yes`


##创建虚拟机

###创建前准备
>把系统镜像（iso）放到某个目录下,这里的系统镜像路径为:`/opt/centos7.iso`

###开始创建虚拟机：
>创建一个存放虚拟机系统的卷：
`qemu-img create -f qcow2 /var/lib/libvirt/images/base.qcow2 50G`

>参数说明：
```
-f							指定卷的格式为qcow2
/var/lib/libvirt/images		虚拟机卷存放的默认路径
base.qcow2					完整卷名
50G							卷的大小
2）创建虚拟机：
virt-install \				#创建命令
-n base \					#虚拟机显示名（非虚拟机主机名）
-r 4096 \					#虚拟机内存大小
--vcpus 2 \					#虚拟机cpu个数
--disk /var/lib/libvirt/images/base.qcow2 \			#系统磁盘卷路径
--location /var/lib/libvirt/images/centos7.iso \		#系统安装iso路径
--nographics \										#不调用图形化界面
--network bridge=br0 \								#网卡1指定网桥
--network bridge=br0 \								#网卡2指定网桥
--os-type linux \									#操作系统类型
--os-variant rhel7 \									#操作系统版本
--console pty,target_type=serial \					#console控制通道
--extra-args 'console=ttyS0,115200n8 serial'			#文本输出

```

###进入centos系统安装

>语言设置：默认英语
>时区设置：选择“Asia”亚洲-“shanghai”
>软件包选择：选择“Infrastructure Server”（基础设施服务器），然后按c继续；
>系统分区
>密码设置
>完成设置后可以开始安装系统了，按“b”开始安装
>按“q”退出，按“b”开始安装。必须以上带“！”号的都必须才能开始安装
>到此安装结束，按回车重启系统

>回到宿主机查看网桥情况，可以看到在“interfaces”下多了2个网卡，vnet0，vnet1: 
`brctl show`


###配置虚拟机：
>设置主机名：`hostnamectl set-hostname base.xftest.com`
>配置IP地址
>>可以使用NetworkManager网络管理工具快速设置：`nmtui`


>退出再次激活网卡即可


>按下回车，重置*号

>查看ip:`ifconfig`

>测试与主机的通信，测试与外界的通信:
```
ping 192.168.80.40
ping www.baidu.com
```

>测试没问题，命令行创建虚拟机完毕！


##虚拟机管理
###添加网卡（在线）：
>查看虚拟机host在物理机上的网卡信息（在物理机操作）:
`virsh domiflist host`

>临时添加网卡:
`virsh attach-interface host --type bridge --source llpbr0`

>命令行增加的网卡只保存在内存中，重启后失效，需要保存到配置文件中
```
virsh dumpxml host > /etc/libvirt/qemu/host.xml
virsh define /etc/libvirt/qemu/host.xml
```

>删除网卡：
`virsh detach-interface host --type bridge --mac [要删除网卡的mac地址]`

###链路聚合（网卡绑定）：
>查看当前网卡信息：`nmcli con show`
>>名称 UUID 类型 设备:
``` 
有线连接 1 44b2b732-48be-3d4c-93e4-59e4a024959e 802-3-ethernet eth2 
有线连接 2 dedad65f-14f6-3961-b3a7-2bbc2f365660 802-3-ethernet ens9 
有线连接 3 c9a3dee1-cbcc-35e7-9f66-e6406ee94cd2 802-3-ethernet ens10 
有线连接 4 b1dfea6c-8e90-35e7-97dc-f467c084fbe6 802-3-ethernet eth0 
eth0 d23c6dde-f39c-42c7-b53f-c09e34de31c9 802-3-ethernet -- 
eth1 4f25cafb-e788-4302-8ba4-8077177e2e15 802-3-ethernet -- 
```

>创建一个名为“team0”并且属性为team的链路接口
nmcli con add type team con-name team0 ifname team0 config '{"runner":{"name":"activebackup"}}'

>为“team0”配置ip地址、网关、DNS,并设置为“手动配置”（manual）
```
nmcli connection modify team0 ipv4.addresses 172.16.20.111/16 
ipv4.gateway 172.16.0.1 ipv4.dns 172.16.20.1 ipv4.method manual
```

>将ens9，ens10网卡 加到“team0”接口:
```
nmcli connection add type team-slave ifname ens9 con-name team-port1 master team0
nmcli connection add type team-slave ifname ens10 con-name team-port2 master team0
```

>启用接口中的网卡:
```
nmcli connection up team-port1
nmcli connection up team-port2
```

>查看team状态:
`teamdctl team0 state view`

>查看其他:
```
ifconfig team0
teamnl team0 ports
teamnl team0 options
```

>>验证：
```
ping 2个网卡的team地址，禁用ens9
nmcli deivce disconnect ens9 
```

###添加磁盘：
>创建磁盘卷disk.qcow2 ,200G
`qemu-img create -f qcow2 /var/lib/libvirt/images/disk01.qcow2 200G`


>vim /etc/libvirt/qemu/disk01.xml
```
<disk type='file' device='disk'>
<driver name='qemu' type='qcow2' cache='none'/>
<source file='/var/lib/libvirt/images/disk01.qcow2'/>
<target dev='vdb' bus='virtio'/>
</disk>

virsh attach-device host disk01.xml --live --persistent
```

###删除磁盘
>分离虚拟机host上的vdb磁盘
`virsh detach-disk host --target vdb` 


##虚拟机克隆

###克隆前准备：
>镜像清理,清理所有操作及残留痕迹，如ssh密钥:
`virt-sysprep --operations ssh-hostkeys.......-a test.qcow2` 

>硬盘启动：
`virt-install 加入参数：--boot hd,menu=on`

>虚拟机克隆：
`virsh# shutdown host #关闭虚拟机host`

>克隆虚拟机host，生成虚拟机new-host
`virt-clone --connect=qemu:///system -o host -n new-host -f /var/lib/libvirt/images/new.qcow2`

###克隆完成后，检查相关配置（mac不可一致，卷路径匹配）：
>确认新虚拟机已生成xml文件：
```
ls -lh /etc/libvirt/qemu/host.xml /etc/libvirt/qemu/new-host.xml
	-rw------- 1 root root 3.2K 6月 27 18:56 /etc/libvirt/qemu/host.xml
	-rw------- 1 root root 3.2K 6月 27 18:33 /etc/libvirt/qemu/new-host.xml
```
>确认新虚拟机与克隆机Mac地址不同:
```
grep "mac address" /etc/libvirt/qemu/host.xml /etc/libvirt/qemu/new-host.xml 

	/etc/libvirt/qemu/host.xml: <mac address='52:54:00:57:31:b7'/>
	/etc/libvirt/qemu/new-host.xml: <mac address='52:54:00:ee:f6:b4'/>

```

>确认新虚拟机source文件路径正确：
```
grep "source file" /etc/libvirt/qemu/host.xml /etc/libvirt/qemu/new-host.xml 

	/etc/libvirt/qemu/host.xml: <source file='/var/lib/libvirt/images/host.qcow2'/>
	/etc/libvirt/qemu/new-host.xml: <source file='/var/lib/libvirt/images/new-host.qcow2'/>

```

>确认新虚拟机卷路径以及卷名正确：
```
ls -lh /var/lib/libvirt/images/host.qcow2 /var/lib/libvirt/images/leon-template.qcow2 

	-rw------- 1 qemu qemu 1.5G 6月 27 19:22 /var/lib/libvirt/images/host.qcow2
	-rw-r--r-- 1 root root 1.5G 6月 27 18:55 /var/lib/libvirt/images/new-host.qcow2

```

###修改主机名
>virsh# shutdown host #关闭虚拟机host

>导出xml文件:
```
cd /etc/libvirt/qemu 
virsh dumpxml host > new-host.xml
```

>编辑new-host.xml，将"test"全部替换 成“new-test”:
`sed -i 's/test/new-test/g' new-test.xml`

>移动虚拟机host，新定义虚拟机new-host:
```
virsh undefine host
virsh define new-host.xml

```

>修改new-host对应卷名:
```
cd /var/lib/libvirt/images
mv host.qcow2 net-host.qcow2

```

##快照功能
>虚拟机管理工具：Virsh
>在宿主机输入virsh，会进入虚拟机管理终端：


>查看当前所有虚拟机（包括关机状态的虚拟机）：`list –all`


###为虚拟机创建快照（最好在关机状态下创建快照）
>关闭虚拟机base

>创建快照：`snapshot-create-as base --name new-install`

>查看快照：`snapshot-list base`

>恢复快照：`snapshot-revert base new-install`


##虚拟机转移
###移动虚拟机系统到虚拟机:virt-v2v
>将目标虚拟机磁盘做到共享存储
`virt-v2v -ic qemu+ssh://172.16.19.200/system test -o local -os /var/tmp -of qcow2 --bridge virbr0`

>参数说明：
```
qemu+ssh://172.16.19.200/system  #连接到目标主机
-o local -os /var/tmp     #把目标虚拟机磁盘数据保存到本地/var/tmp目录
-of qcow2       #以qcow2格式保存
--bridge virbr0      #指定网桥
```

>成功后会新生成2个文件：`test-sda test-xml`

##热迁移（在线迁移）
###存储卷在共享存储池外的虚拟机迁移

>环境：
```
宿主机A（图形）：172.16.19.100   虚拟机：host
宿主机B（图形）：172.16.19.200
需要把host 从宿主机A迁到宿主机B
共享池地址：172.16.19.1/var/nfs
```

>命令行热迁移： --live 在线迁移 --unsafe 支持不稳定迁移 qemu+ssh 目标主机:
`virsh migrate --live --unsafe host qemu+ssh://172.16.19.200/system`

###在宿主机A上：
>打开virsh-manager，连接宿主机B，在A、B上建立共享存储：
>把虚拟机host存储卷copy到本地映射的共享存储位置
