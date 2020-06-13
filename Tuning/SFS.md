#挂载SFS文件系统

##实验目的
>理解华为云存储服务的基本原理；

##实验内容
>创建SG（安全组），购买鲲鹏云ECS弹性云主机；
>购买鲲鹏云相关云存储服务：
```
	a.弹性文件服务
	b.对象存储服务OBS
	c.云硬盘
```

>云存储服务的配置与使用
```
	a.ECS挂载文件服务
	b.上传/下载文件于OBS服务
	c.挂载云硬盘
```

##实验拓扑图
![](https://github.com/Becky-nuo/git-test/blob/master/images/SFS/001.png)

##实验步骤
>登录华为云，创建SG（安全组），购买云主机

###创建SG
>点击“控制台”-“虚拟私有云VPC”-“访问控制”-“安全组”

>点击右上角的“创建安全组”；

>创建安全组-配置规则：
>>为了更好的测试连通性，我们对入口方向放行ping协议ICMP；
>>为了可以使用远程工具登录ECS，我们对入口方向放行SSH协议；


###购买一台ECS弹性云主机
>进入服务列表，选择“弹性云主机”：
>点击右上角“购买弹性云主机”：
>选择云主机的配置（此实验性能方面要求不高，建议都选低配）
>配置清单：
```
1.按需收费
2.通用计算型-s6.small.1
3.公共镜像-CentOS-Centos7.6 64bit（40GB）
4.通用型SSD

网络配置：
1.选中一个VPC，单网卡
2.选中前面创建的安全组：XF-sfs
3.“现在购买”弹性公网IP
4.静态BGP
5.按带宽计费
6.2M带宽





自定义购买配置，完成后点击“购买”
1.设置ECS服务器名称：XF-video
2.设置root密码：huawei@123
3.暂不购买云备份
```

###通过公网弹性IP，使用远程工具登录（X-Shell）
>先测试能否ping （安全组XF-sfs入口方向已放行ICMP协议);
>使用root登录（安全组XF-sfs入口方向已放行SSH协议）;
>输入root密码huawei@123，登录成功！


###购买相关云存储服务
>购买“弹性文件服务”,点击“服务列表”-“存储”-“弹性文件服务”,点击右上角的“创建文件系统”;
>选项：
```
选择文件系统类型：SFS
区域：当前ECS所在区域
可用区：当前ECS所在可用区域
协议类型：NFS
虚拟私有云：当前所在VPC
自动扩容：是
名称：XF-sfs
购买量：1
```


###使用ECS挂载刚登录的文件系统
>登录ECS，安装nfs组件包:
`yum install nfs-utils –y`


>新建挂载目录：/XF-nfs
`mkdir /Xf-nfs`


>使用命令以下命令，将云文件系统挂载到/XF-nfs目录下
```
mount -t nfs -o vers=3,timeo=600,nolock,rsize=1048576,wsize=1048576,hard,retrans=3,noresvport,async,noatime,nodiratime sfs-nas1.cn-south-1c.myhuaweicloud.com:/share-08133236 /XF-nfs
```
红色字体为云文件系统的挂载地地，可以弹性文件服务列表中获得



>查看挂载情况：
`mount |grep XF-nfs`


>设备ECS云主机开机自动挂载云文件系统：
编辑自动挂载文件，写入以下内容：
```
vim /etc/fstab
sfs-nas1.cn-south-1c.myhuaweicloud.com:/share-08133236 /XF-nfs nfs vers=3,timeo=600,nolock,rsize=1048576,wsize=1048576,hard,retrans=3,noresvport,async,noatime,nodiratime 0 0
```

>保存退出后，使用mount -a命令检查挂载是否有错，无结果报错则ok;


###云对象存储服务（OBS）的使用
>在“服务列表”中选中“对象存储服务OBS”,点击右上角的“创建桶”;
>设置：
```
桶名：xf-obs
存储类别：标准存储
桶策略：公共读
```

>创建完成后，点击桶名-对象，尝试上传一个视频压缩文件video.zip

>添加文件，上传video.zip
>>上传成功！


>点击“更多”选项中的“复制对象URL”可以获得下载地址


>使用ECS云主机下载这个文件到挂载目录/XF-nfs，这时的效果其实就相当于把文件存放到了云存储文件系统中了,登录ECS云主机使用以下命令下载：
`wget -O /XF-nfs/video.zip 对象下载URL ` 

>查看下载情况：ls /XF-nfs



###云硬盘的使用
>购买云硬盘,在“服务列表”中选择“云硬盘”,点击右上角，“购买磁盘”;
>硬盘配置：
```
计费模式：按需收费
可用区：ECS所在可用区
磁盘规格：通用型SSD
```
>返回磁盘列表，可以看到新购买的磁盘，点击“挂载”，选中挂载的ECS主机即可

>登录ECS主机，使用命令lsblk查看磁盘挂载情况：`lsblk`

>对新磁盘进行格式化后即可正常使用:`mkfs.ext4 /dev/vdb`