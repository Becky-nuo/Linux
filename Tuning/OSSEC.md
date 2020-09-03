# OSSEC 实验
> 服务端（192.168.204.10），客户端（192.168.204.20）

## 服务端
> 配置系统环境，下载依赖
```
	1. yum install git -y 
	2. setenforce 0 
	3. systemctl stop firewalld 
	4. systemctl disable firewalld 
	5. yum install -y mariadb-server mariadb mariadb-devel 6. vi /etc/selinux/config

```
![关闭控制节点](https://github.com/Becky-nuo/git-test/blob/master/images/OSSEC/003.png)

> 启动数据库
```
	1. systemctl start mariadb 
	2. systemctl enable mariadb 
	3. mysql_secure_installation 
	4. mysql -uroot -proot
```

> 设置数据库
```
	1. create database ossec;
	2. grant all on ossec.* to ossec@localhost; 
	3. set password for ossec@localhost=password(‘ossec‘); 4. flush privileges; 
	5. exit
```

> 下载OSSEC,添加OSSEC表结构 
```
	1. git clone https://github.com/ossec/ossec-hids.git 
	2. cd ossec-hids/src/os_dbd/ 
	3. mysql -uossec -p ossec < mysql.schema
```

> 安装docker 
```
	1. yum install -y yum-utils device-mapper-persistent-data 
	2. yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo 
	3. yum install docker-ce
```

> 安装OSSEC
```
	1. systemctl start docker.service 
	2. docker run --name ossec-server -d -p 1514:1514/udp -p 1515:1515 -e SYSLOG_FORWADING_ENABLED=true -e SYSLOG_FORWARDING_SERVER_IP=192.168.204.10 - v /var/ossec/data:/var/ossec/data xetusoss/ossec-server
```

> 检测docker运⾏情况  
	`docker ps`
![](https://github.com/Becky-nuo/git-test/blob/master/images/OSSEC/001.png)

> 进⼊docker  
 `docker exec -it ossec-server bash`
![](https://github.com/Becky-nuo/git-test/blob/master/images/OSSEC/002.png)

> 对服务端进⾏设置  
```
	1. /var/ossec/bin/ossec-control enable database 
	2. chmod u+w /var/ossec/etc/ossec.conf 
	3. apt install vim 
	4. vi /var/ossec/data/etc/ossec.conf

```

> 报错
![](https://github.com/Becky-nuo/git-test/blob/master/images/OSSEC/004.png)

> 解决
![](https://github.com/Becky-nuo/git-test/blob/master/images/OSSEC/005.png)

## 配置客户端 
> 安装客户端： 
```
	1. wget https://github.com/ossec/ossec-hids/archive/3.6.0.tar.gz 
	2. tar -xvf ossec-hids-3.6.0.tar.gz 
	3. yum install zlib-devel pcre2-devel make gcc zlib-devel pcre2-devel sqlite- devel openssl-devel libevent-devel 
	4. ./install 
```

> 添加KEY 
```
	1 sudo touch /var/ossec/queue/rids/sender 
	2 /var/ossec/bin/manage_agents 

```

> 输⼊上⾯在服务端获取的KEY 
![]()
> 启动客户端 
`/var/ossec/bin/ossec-control start`


## 查看连接情况 

> 在服务端的docker中输⼊  
`/var/ossec/bin/agent_control -l`
