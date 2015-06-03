##nagios##

###前言###
nagios监控


###安装###
	# http://pkgs.repoforge.org/rpmforge-release/
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

	#安装plugin
	 wget http://nagios-plugins.org/download/nagios-plugins-2.0.3.tar.gz
	./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-command-user=nagios --with-command-group=nagcmd --prefix=/usr/local/nagios
	make all
	make install
	chmod 755 /usr/local/nagios
	
	chkconfig httpd on
	chkconfig nagios on
	service httpd restart
	service nagios restart


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
	useradd -s /sbin/nulgin nagios
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


	#设置服务器地址
	vi /usr/local/nagios/etc/nrpe.cfg
	找到 allowed_hosts=127.0.0.1
	后面加nagios服务器的IP, 用“,”隔开，加了之后如下：
	allowed_hosts=127.0.0.1,192.168.100.17

	#启动nrpe
	/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
	lsof -i:5666
	
	#服务端测试手段
	libexec/check_nrpe -H vm17  -c check_load
	NRPE v2.15

	
###添加监控项###

监控项在客户端添加

	vim  /etc/local/nagios/etc/nrpe.cfg
	
	command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10% 
	command[check_disk_/]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/sda2
	command[check_disk_web]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/mapper/vg_21tb-DATA

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
        command_line    $USER1$/check_nrpe -H  $HOSTNAME$ -c $ARG1$
        }

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
		
	
	