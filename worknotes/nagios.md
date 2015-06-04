##nagios##

###前言###
Nagios是一款用于系统和网络监控的应用程序。它可以在你设定的条件下对主机和服务进行监控，在状态变差和变好的时候给出告警信息。参考中文手册：http://nagios-cn.sourceforge.net/nagios-cn/bookfirst.html


###安装###
安装nagios-core，官网下载地址 http://pkgs.repoforge.org/rpmforge-release/ ，首先要更新下rpmforge，安装些基础编码依赖包

	rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el7.rf.x86_64.rpm
	yum install gd fontconfig-devel libjpeg-devel libpng-devel gd-devel perl-GD \
	openssl-devel php mailx postfix cpp gcc gcc-c++ libstdc++ glib2-devel  libtoul-ltdl-devel 

	#add user and group
	groupadd -g 6000 nagios 
	groupadd -g 6001 nagcmd 
	useradd -u 6000 -g nagios -G nagcmd -d /home/nagios -c "Nagios Admin" nagios

	#https://www.nagios.org/download/core/thanks
	wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-4.0.8.tar.gz
	tar xzfv nagios-4.0.8.tar.gz 
	cd nagios-4.0.8 
	./configure --prefix=/usr/local/nagios --with-nagios-user=nagios \ 
	--with-nagios-group=nagios --with-command-user=nagios 
	--with-command-group=nagcmd --enable-event-broker --enable-nanosleep  
	--enable-embedded-perl --with-perlcache     
	make all            
	make install         
	make install-init      
	make install-commandmode   
	make install-webconf    
	make install-config   

服务器选择，nagios需要php解析服务器，文件路劲在/usr/local/nagios/share下，采用apache httpd 加php解析比较简单

	#http php
	yum install httpd php* 

	#配置httpd.conf,支持php
	vim /etc/httpd/conf/httpd.conf
	DirectoryIndex index.html index.html.var 
	将其修改为：
	
	DirectoryIndex index.html index.php 
	再在 Apache 配置文件下增加如下内容：
	
	AddType application/x-httpd-php .php 


	#设置用户访问控制
	htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin 
	chown nagios:nagcmd /usr/local/nagios/etc/htpasswd.users 
	usermod -a -G nagios,nagcmd apache 
	service httpd restart  

	#Postfix
	chkconfig postfix on
	service postfix start

采用插件方式可以自定义监控，很容易上手，由于服务端只是调用nrpe,因此make install-plugin可以调用调试即可。

	#安装plugin
	 wget http://nagios-plugins.org/download/nagios-plugins-2.0.3.tar.gz
	./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-command-user=nagios --with-command-group=nagcmd --prefix=/usr/local/nagios
	make all
	make install
	chmod 755 /usr/local/nagios

	#安装nrpe http://sourceforge.net/projects/nagios/files/nrpe-2.x/
	wget http://sourceforge.net/projects/nagios/files/nrpe-2.x/nrpe-2.15/
	./configure --with-command-group=nagios --prefix=/usr/local/nagios 
	make all
	make install-plugin

	chkconfig httpd on
	chkconfig nagios on
	service httpd restart
	service nagios restart


让nagios支持绘图，需要安装pnp，由于没使用，暂时作为参考。

	#安装rrdtool，pnp依赖rrd  http://oss.oetiker.ch/rrdtool/pub/?M=D
	yum -y install libxml2-devel pango-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker
	wget http://oss.oetiker.ch/rrdtool/pub/rrdtool-1.5.3.tar.gz
	./configure --prefix=/usr/local/rrdtool
	make all
	make install


	#安装pnp http://sourceforge.net/projects/pnp4nagios/	
	yum -y install  perl-Time-HiRes
	wget http://sourceforge.net/projects/pnp4nagios/files/latest/download
	./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-rrdtool=/usr/local/rrdtool/bin/rrdtool --with-perfdata-dir=/usr/local/nagios/share/perfdata
	make all
	make install
	make install-config
	make install-init
	
	

###客户端安装###
被监控端安装nagios-plugins和nrpe

	#添加使用账户
	useradd -M -s /sbin/nulgin nagios
	yum -y install openssl-devel

	#安装nagios-plugins
	./configure
	make && make install

	#安装nrpe http://sourceforge.net/projects/nagios/files/nrpe-2.x/
	wget http://sourceforge.net/projects/nagios/files/nrpe-2.x/nrpe-2.15/
	./configure --with-command-group=nagios --prefix=/usr/local/nagios 
	make all
	make install-plugin
	make install-daemon
	make install-daemon-config


设置服务器地址

	vi /usr/local/nagios/etc/nrpe.cfg
	找到 allowed_hosts=127.0.0.1，后面加nagios服务器的IP, 用“,”隔开，加了之后如下：
	allowed_hosts=127.0.0.1,192.168.100.17
	如果服务器使用nat,真实访问外部机器的是网关那台机器的IP地址，因此allowed_hosts应加上网关IP地址。

	#启动nrpe
	/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
	lsof -i:5666
	
	#服务端测试手段
	libexec/check_nrpe -H vm14  -c check_load
	NRPE v2.15

	
###添加监控项###

监控项在客户端添加，常规检测报警参数，w--warning,c--critic,u--unkown,r--recovery。

	vim  /etc/local/nagios/etc/nrpe.cfg
	
	command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10% 
	command[check_disk_/]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /
	command[check_disk_web]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /web
	
解释上条命令，意思为：检测此磁盘，free<20%，就warning,free<10%,就critic。具体报警要参考对应检测服务选择的模板，以自带generic-service模板为例，截取部分模板内容：

	check_period                    24x7                   
	max_check_attempts              3                     
	normal_check_interval           10                     
	retry_check_interval            2                      
	contact_groups                  admins               
	notification_options            w,u,c,r               
	notification_interval           60                    
	notification_period             24x7             

解释为：

###服务端配置###

使用独立配置
	
	cd /usr/lcoal/nagios/etc
	cp -r objects  monitor
	
修改nagios.cfg，使用cfg_dir添加配置目录，注释objects中重复项

	cfg_dir=/usr/local/nagios/etc/monitor
	



利用nrpe收集数据
>在command.cfg中添加使用nrpe的命令

	define command{
        command_name    check_nrpe
        command_line    $USER1$/check_nrpe -H  $HOSTADDRESS$ -c $ARG1$
        }
	
	#鉴于犯过参数写为$HOSTNAME$，主机无法解析主机名，一直bound报错，特此备注警告！

>新建hosts.cfg，创建自己的监控节点
	
	define host{
		check_command		check-host-alive
		notification_options	d,u,r
		max_check_attempts 	5
		name			generichosttemplate
		register		0
		contact_groups		users
		}
	
	define host{
		host_name		vm15
		address			192.168.100.15
		use			linux-server
		}
	
	define host{
		host_name		vm14
		address			192.168.100.14
		use			linux-server
		}
	
	define host{
		host_name		vm16
		address			192.168.100.16
		use			linux-server
		}
	define hostgroup{
		hostgroup_name		vm
		alias			this is vm host group
		members			vm14,vm15,vm16
		}
	
>新建services.cfg，添加自定义监控服务列表

	define service{
		use				generic-service
		hostgroup_name			vm
		service_description		swap
		check_command			check_nrpe!check_swap
		}
	
	
	define service{
		use				generic-service
		hostgroup_name			vm
		service_description		load
		check_command			check_nrpe!check_load
		}
	
	define service{
		use				generic-service
		hostgroup_name			vm
		service_description		users
		check_command			check_nrpe!check_users
		}
	
	
	define service{
		use				generic-service
		hostgroup_name			vm
		service_description		disk_/
		check_command			check_nrpe!check_disk_/
		}
	
	
	define service{
		use				generic-service
		hostgroup_name			vm
		service_description		disk_web
		check_command			check_nrpe!check_disk_web
		}
	
	
	define servicegroup{
		servicegroup_name		linux_disk_services
		alias				linux server disk check
		members				vm15,disk_/,vm15,disk_web
		}


服务端测试获取

	# /usr/local/nagios/libexec/check_nrpe -H vm15 -c check_disk_/
	DISK OK - free space: / 27143 MB (93% inode=96%);| /=1782MB;24378;27425;0;30473
		
		
###娱乐版###
基于clush的批量安装nagios客户端，由于爆了很多warning，就不介绍了。

	# ls /root/nagios/
	nagios_client.sh  nagios-plugins-2.0.3.tar.gz  nrpe-2.15.tar.gz  nrpe.cfg  nrpe_restart

	clush -b -w @hadoop -c /root/nagios  --dest=/web
	clush -b -w @hadoop /web/nagios/nagios_client.sh

	#服务端测试
	/usr/local/nagios/libexec/check_nrpe -H HSlave1 -c check_disk_/

>nagios_client.sh 

	#!/bin/sh
	
	echo "check nagios user nagios..."
	grep nagios /etc/passwd  1>/dev/null 2>/dev/null
	if [ $? -ne 0 ];then
		echo "Now add user nagios"
		useradd -M -s /sbin/nologin nagios
	fi
		
	echo "check gcc rpm..."
	rpm -q gcc 1>/dev/null 2>/dev/null
	
	if [ $? -ne 0 ];then
		echo " Now yum -y install gcc..."
		yum -y install gcc
	fi
	
	echo "check openssl rpm..."
	rpm -q openssl-devel 1>/dev/null 2>/dev/null
	
	if [ $? -ne 0 ];then
		echo "Now yum -y install openssl-devel.."
		yum -y install openssl-devel
	fi
	
	cd /web/nagios
	tar zxf nagios-plugins-2.0.3.tar.gz
	cd nagios-plugins-2.0.3
	./configure 
	make && make install
	
	
	cd ..
	
	tar zxf  nrpe-2.15.tar.gz
	cd nrpe-2.15
	./configure --with-command-group=nagios --prefix=/usr/local/nagios
	
	make all
	make install-plugin
	make install-daemon
	make install-daemon-config
	 
	cp /web/nagios/nrpe.cfg  /usr/local/nagios/etc
	
	mkdir /root/bin
	cp /web/nagios/nrpe_restart /root/bin
	chmod +x /root/bin/nrpe_restart
	sh /root/bin/nrpe_restart 

>nrpe_restart

	#!/bin/sh

	pid=`cat /var/run/nrpe.pid `
	if [ -d /proc/$pid -a /var/run/nrpe.pid ];then
		echo "Now kill nrpe..."
		kill -9 $pid
	fi
	
	sleep 1
	
	echo "Now start nrpe..."
	/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d