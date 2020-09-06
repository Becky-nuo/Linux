# SSH服务和TCP Wrappers实验 

## 实验环境： 
> ⼀台Linux服务器A（ssh client） 
> ⼀台Linux服务器B（ssh server） 

## 实验步骤： 

>确保A、B服务器可以通讯。

>step1：在服务器A上⽤公钥⽣成⼯具⽣成公钥
`ssh-keygen -t rsa` 

>> 先直接按回⻋使⽤默认的key⽣成路径，在下图步骤输⼊私钥：12345678

>> 设置成功：

>step2：在服务器B创建新⽤户Server，在⽤户⽬录下创建.ssh⽂件夹，同时修改成与服务器A的.ssh⽂件 夹相同的权限，再将服务器A⽣成的公钥⽂件传给服务器B。 

>> 服务器B：
`mkdir .ssh 2 chmod 755 .ssh` 

>> 服务器A： 
`scp .ssh/id_rsa.pub Server@192.168.56.102:/home/Server/.ssh/authorized_keys`

> step3：确认公钥⽂件 
>> 服务器A： 
>> 服务器B：

>step4：修改服务器B的ssh配置 
`sudo vi /etc/ssh/sshd_config` 
>>确认下列属性设置正确 
`AuthorizedKeyFile .ssh/authorized_keys PasswordAuthentication no`

> step5：在服务器A⽤root⽤户通过ssh登陆服务器B的Server⽤户
>vstep6：在服务器A创建新⽤户test
>vstep7：使⽤test⽤户在服务器A⽤ssh登陆服务器B的Server⽤户被拒绝登陆
>step8：服务器B中确认ssh⽀持TCP Wrappers

>step9：服务器B中屏蔽ssh服务
`vi /etc/hosts.deny`

>step10：再次尝试登陆
>step11：在服务器B中复制root⽬录下的.ssh/到test⽤户⽬录下，将sshd服务解禁，再次尝试登陆（注 意.ssh⽂件夹的权限）

>step12：在服务器B上下载DenyHosts 
```
	wget 'https://sourceforge.net/projects/denyhosts/denyhosts/files/latest/download'
	unzip DenyHosts.zip
```

>step13：安装DenyHosts
```
	cd denyhosts-2.10 
	python setup.py install 
```

>>修改denyhosts.cfg的SECURE_LOG选项为redhat 
```
	cp denyhosts.cfg /etc/ 
	cp daemon-control-dist daemon-control
	sudo ln daemon-control /etc/init.d/
```
>>修改daemon-control的内容：

>step14：开启ssh密码登陆后在服务器A通过ssh登陆服务器B的root⽤户 这时候再看服务器B的/etc/hosts.deny