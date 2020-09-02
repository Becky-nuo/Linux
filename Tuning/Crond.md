# Crond+Shell系统备份管理

## 项目要求

 >在需要备份数据的服务器端（客户端）： 
```
1. 需要备份数据的服务器每天凌晨1点自动在本地打包备份数据（备份内容按照具体服务器要求确定）; 
2. 备份数据需存放在以“主机名_IP地址”命名的目录中（例如：NFS_192.168.99.101）; 
3. 每天备份的数据文件名称为“主机名_IP地址_当天日期”（例如：NFS_192.168.99.101_2018-11-25）; 
4. 本地打包完成后，自动调用rsync将备份数据包推送到备份服务器; 
5. 为了避免空间浪费，本地仅保留最近7天的备份数据包; 
```

>在备份服务器端（服务端）： 
```
1. 备份服务器端部署rsync，用于接收客户端推送过来的备份数据; 
2. 备份服务器端需要每天凌晨5点自动开始校验客户端推送过来的数据是否完整; 
3. 备份服务器端在校验完数据后自动通过邮件将校验结果通知给管理员; 
4. 为了保证备份数据的可用性同时避免空间浪费，备份服务器仅保留最近180天的备份数据包;
```
## 背景知识

### 数据打包工具-tar

#### tar—基本用法：
>命令格式： • tar 选项 压缩包的名字 被压缩的源文件；

>常用选项: 
```
-c:	创建归档 打包 ； 
-x:	释放归档 解包； 
-f:	指定归档文件名称；  
-v：	输出详细信息；
```

>压缩格式 :
```
.gz 传统的压缩格式，压缩速度快 ---> gzip、gunzip -z 
.bz2 较新，压缩比例高一些 ---> bzip2、bunzip2 -j
.xz 最新，压缩比例高、压缩效率快 ---> xz、unxz -J
```

#### tar—备份基本用法范例：

>使用gz格式打包/etc 目录:
`tar czvf etc.tar.gz /etc`

>使用bz2格式打包/etc 目录:
`tar cjvf etc.tar.gz /etc`

#### tar—恢复基本用法范例:

>gz格式数据包恢复:
`tar xzvf etc.tar.gz -C /` 

>bz2格式数据包恢复:
`tar xjvf etc.tar.gz -C /`

>备份/etc
`tar czvf etc.tar.gz /etc`

>模拟yum配置文件损坏
`rm /etc/yum.conf`

>检验yum是否能正常工作
`yum list `

>恢复/etc目录
`tar xzvf etc.tar.gz -C /`

>再次检验yum是否能正常工作:`yum list`


## 创建备份目录

>定义变量: 
```
Host=$(hostname) 
Addr=$(hostname -I) 
Date=$(date +%F) 
DestDir=${Host}_${Addr} 
BackupRoot=/backup 
```

>创建备份目录 :
`[ -d $BackupRoot/$DestDir ] || mkdir -p $BackupRoot/$DestDir`


## 备份文件 

>备份对应的文件: 
`cd ${BackupRoot}/${DestDir} && tar czf ${Host}_${Date}_etc.tar.gz /etc`

## 生成校验码 
>生成MD5验证信息 :
`cd ${BackupRoot}/${DestDir} && md5sum ${Host}_${Date}_etc.tar.gz > ${Host}_${Date}_etc.tar.gz.md5 `

>校验方法:
`md5sum -c nfs-server_2018-11-25_etc.tar.gz.md5 `

>模拟破坏数据
`echo 1111 >> nfs-server_2018-11-25_etc.tar.gz.md5 `

>再次校验
`md5sum -c nfs-server_2018-11-25_etc.tar.gz.md5`

## 推送本地数据至备份服务器
 
>推送本地数据至备份服务器 :
`export RSYNC_PASSWORD=1 rsync -av $BackupRoot/ rsync_backup@192.168.99.102::backup`

##本地保留最近7天的数据 

>模拟生成1-25日旧备份数据 :
`for i in {1..25};do date -s 2018/11/$i && sh /root/MyBackup.sh;done `

>本地保留最近7天的数据 :
`find $BackupRoot/ -type f -mtime +7 | xargs rm -rf`

>Crond配置定时任务
``` 
crontab -e [-u username] 
* * * * * command 
分 时 日 月 周 命令 
0 1 * * * sh /root/MyBackup.sh
```

## 部署其他需备份的服务器 

>设置主机名:
 `hostnamectl set-hostname web1 `

>安装:
`rsync ü yum install rsync `

>下载备份脚本:
`rsync -avz root@192.168.99.101:/root/MyBackup.sh . `

>设置定时任务
`crontab -e`


## 服务端校验数据
 
### 配置邮件发送端 
>安装邮件客户端 :`yum install mailx -y `

>配置smtp发送参数 
```
vim /etc/mail.rc 

	set from=1234568@qq.com 
	set smtp="smtps://smtp.qq.com:465" 
	set smtp-auth-user=123456 set smtp-auth-password=xxxxxxx (邮箱账号的客户端授权码，需要登陆自己的 邮箱进行设置，不是邮箱密码) 
	set smtp-auth=login 
	set ssl-verify=ignore 
	set nss-config-dir=/etc/pki/nssdb
```
### 编写脚本 
>定义变量 :
```
	BackupRoot=/backup 
	Date=$(date +%F) 
```

>遍历目录，校验数据 
```
for DataDir in $(ls ${BackupRoot}) 
do 
	cd ${BackupRoot}/${DataDir} 
	find -type f -name *.md5 | xargs md5sum -c >> /root/CheckResult_${Date} 
done
```

>发送邮件 ：
`mail -s "Rsync Backup $Date" username@mail.com < /root/CheckResult_${Date} `

>删除180天以上备份： 
`find ${BackupRoot} -type f -mtime +180 | xargs rm -rf`

>配置crontab，每天早上5点校验备份数据:
``` 
crontab -e -u root 
0 5 * * * /DirName/ScriptName
```
