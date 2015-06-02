##nagios##

###前言###



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
	
	
	
	