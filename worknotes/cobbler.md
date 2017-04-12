## cobbler

### 前言

Cobbler是一个快速网络安装linux的服务，而且在经过调整也可以支持网络安装windows。该工具使用python开发，小巧轻便（才15k行代码），使用简单的命令即可完成PXE网络安装环境的配置，同时还可以管理DHCP，DNS，以及yum包镜像。


### 安装

>配置yum源

	#64位机器
	rpm -ivh 'http://mirrors.hust.edu.cn/epel//6/x86_64/epel-release-6-8.noarch.rpm'
	#32位机器
	http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm

>安装依赖服务

	yum -y install dhcp tftp rsync xinetd httpd
	yum -y install cobbler

>启动检测

	service cobblerd start
	service httpd start
	
	cobbler check

		Traceback (most recent call last):
		  File "/usr/bin/cobbler", line 36, in <module>
		    sys.exit(app.main())
		  File "/usr/lib/python2.6/site-packages/cobbler/cli.py", line 655, in main
		    rc = cli.run(sys.argv)
		  File "/usr/lib/python2.6/site-packages/cobbler/cli.py", line 270, in run
		    self.token         = self.remote.login("", self.shared_secret)
		  File "/usr/lib64/python2.6/xmlrpclib.py", line 1199, in __call__
		    return self.__send(self.__name, args)
		  File "/usr/lib64/python2.6/xmlrpclib.py", line 1489, in __request
		    verbose=self.__verbose
		  File "/usr/lib64/python2.6/xmlrpclib.py", line 1253, in request
		    return self._parse_response(h.getfile(), sock)
		  File "/usr/lib64/python2.6/xmlrpclib.py", line 1392, in _parse_response
		    return u.close()
		  File "/usr/lib64/python2.6/xmlrpclib.py", line 838, in close
		    raise Fault(**self._stack[0])
		xmlrpclib.Fault: <Fault 1: "<class 'cobbler.cexceptions.CX'>:'login failed'">

	service cobblerd restart
	cobbler check  --提示如何修改

>check提示项：

	1,编辑/etc/cobbler/settings文件，找到 server选项，修改为适当的ip地址，本实例配置ip为：192.168.100.82

	2，编辑/etc/cobbler/settings文件，找到 next_server选项，修改为适当的ip地址，本实例配置ip为：192.168.100.82

	3，编辑/etc/xinetd.d/rsync文件，将文件中的disable字段的配置由yes改为no
	
	4，提示说debmirror没安装。如果不是安装 debian之类的系统，此提示可以忽略，如果需要安装，下载地址为：
	
	http://rpmfind.net/linux/rpm2html/search.php?query=debmirror
	
	5，ksvalidator没有被发现，安装pykickstart
		yum install pykickstart
	
	6，修改cobbler用户的默认密码，可以使用如下命令生成密码，并使用生成后的密码替换/etc/cobbler/settings中的密码。生成密码命令： 其中“random-phrase-here”为干扰码
	
	openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'
	#选项是数字1，而不是字母l

	7，fencing tools为找到安装
	
	yum install fence-agents
	
	

>导入镜像

	mkdir /mnt/Centos6.5
	mount -o loop CentOS-6.5-x86_64-bin-DVD1.iso /mnt/Centos6.5/
	
	#--name,--arch唯一标示镜像，不可重复
	cobbler import --path=/mnt/Centos6.5 --name=CentOS6.5 --arch=x86_64

	#查看导入源库列表
	cobbler distro list

>配置dhcp服务

	vim /etc/cobbler/settings
	manage_dhcp: 1
	#此选项开启，将使用/etc/cobbler/dhcp.template --> /etc/dhcp/dhcpd.conf。根据需要修改即可

>重启xinetd

	xinetd控制tftp，出现tftp找不到，考虑重启

>同步cobbler

	cobbler sync

	#出现找不到syslinux下的pxelinux.0，menu.c32等
	#yum -y instlal syslinux
	#dhcp失败，查看配置文件，修改/etc/init.d/dhcpd中的user,group	

>客户端测试安装


>客户端指定重装
	
	rpm -ivh 'http://mirrors.hust.edu.cn/epel//6/x86_64/epel-release-6-8.noarch.rpm'
	yum install koan
	
	#查看cobbler上的系统列表
	koan --server=192.168.100.82 --list=profiles
	
	#选择操作系统安装
	koan --server=192.168.10.1 --profile=CentOS6.5-x86_64 --replace-self
	
### cobbler_web

>安装

	yum -y install cobbler-web
	
>修改或添加用户

	htdigest /etc/cobbler/users.digest "Cobbler" cobbler  

>配置cobbler_web可以登录

	sed -i 's/authn_denyall/authn_configfile/g' /etc/cobbler/modules.conf

>重启cobber和http

	service cobblerd restart
	service httpd restart

>访问

	http://192.168.100.82/cobber_web

	