##Cacti##

***
###前言###
Cacti 在英文中的意思是仙人掌的意思，Cacti是一套基于PHP、MySQL、SNMP及RRDTool开发的网络流量监测图形分析工具。它通过snmpget来获取数据，使用 RRDtool绘画图形，它的界面非常漂亮，能让你根本无需明白rrdtool的参数能轻易的绘出漂亮的图形。


###Cacti安装###

####安装LAMP环境####

	rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
	ntpdate 202.120.2.101
	yum install -y httpd php php-mysql php-snmp php-xml php-gd mysql mysql-server gd gd-devel

	.....

###httpd配置###

	cat /etc/httpd/conf/httpd.conf
	<VirtualHost *:80>
	  DocumentRoot /web/vhosts/cacti
	  ServerName cacti.test.com
	  ErrorLog logs/cacti.test.com-error_log
	  CustomLog logs/cacti.test.com-access_log common
	  <Directory "/web/vhosts/cacti">
	    Options Indexes FollowSymLinks
	    DirectoryIndex index.php index.html index.htm
	    AllowOverride None
	    Order allow,deny
	    Allow from all
	  </Directory>
	</VirtualHost>




###模板资源###
官方地址为 http://docs.cacti.net/templates，默认规则大致为：
>体积大小4K的，属于模板。 console->Import Templates->import
>
>体积小小4k的，属于配置。 参考路径为：/web/vhosts/cacti/resource/snmp_queries/disk_io.xml


###配置思路简单总结###
	1.下载模板，导入模板和配置文件

	2.Host Templates->add->centos_1
		Associated Graph Templates
		0) ucd/net - CPU Usage	Delete
		0) ucd/net - Load Average	Delete
		0) ucd/net - Memory Usage

		Associated Data Queries
		0) SNMP - Get Disk IO	Delete
		0) SNMP - Get Mounted Partitions	Delete
		0) SNMP - Interface Statistics

	3.console->Devices->Add  
		Host Template ->centos_1

	4.New Graphs->Host: app13	
		Graph Template Name : all 
		Data Query [SNMP - Get Disk IO]
			Index -- dm-0 代表逻辑分区，ll /dev/mapper/** 查看
		Data Query [SNMP - Get Mounted Partitions]
			Index -- 选择监控磁盘
		Data Query [SNMP - Interface Statistics]
			Index -- 选择外网网卡，graph type：In/Out Bits (64-bit Counters)

	5.Graph Trees->Default Tree->Add
		Parent Item -- 父目录
		Tree Item Type -- Header:目录，Host:节点


熟悉流程后，使用脚本批量添加，脚本为 [AddCacti](../giftscript/AddCacti) 
列表名字为：cacti.list，内容如下：

	[root@cacti bin]# cat cacti.list 
	vm14-192.168.100.14
	vm15-192.168.100.15
	vm16-192.168.100.16
	...

	[root@cacti bin]# ./add_cacti.sh 
	Now add device vm14 ...
	Adding vm14 (192.168.100.14) as "centos_1" using SNMP v2 with community "public"
	Success - new device-id: (34)
	Now add graphs ...
	Graph Added - graph-id: (192) - data-source-ids: (339, 339)
	Graph Added - graph-id: (193) - data-source-ids: (340, 341, 342)
	Graph Added - graph-id: (194) - data-source-ids: (343, 344, 345)
	Graph Added - graph-id: (195) - data-source-ids: (346, 347, 348)
	Graph Added - graph-id: (196) - data-source-ids: (349, 349)
	Graph Added - graph-id: (197) - data-source-ids: (350, 350)
	Graph Added - graph-id: (198) - data-source-ids: (351, 351)
	Graph Added - graph-id: (199) - data-source-ids: (352, 352)
	Added Node node-id: (31)
	Now add device vm15 ...


	
###TROUBLESHOOT###
初始登录失败

	#查看mysql编码
	mysql> SHOW VARIABLES LIKE 'collation_%';
	mysql> SHOW VARIABLES LIKE 'character_set_%';
	mysql> set names "latin1"；
	#对比密码
	mysql> select * from user_auth;
	mysql> select md5("admin")；
	#鉴于没成功，推倒重来才行，就只做参考


监控mem不出图

	1 .Console—〉Data Templates默认最大内存是10G，小于服务器内存限制，改为100G。
		rrdtool info localhost_mem_free_1.rrd --查询
	2. 修改以生成的rra文件
		# rrdtool tune *_mem_free_*.rrd -a mem_free:100000000
		# rrdtool tune *_mem_buffers_*.rrd -a mem_buffers:100000000
		# rrdtool tune *_mem_cache_*.rrd -a mem_cache:100000000

rra下没有rrd文件
	
	加入定时任务
	# crontab -u cactiuser -e  
	*/5 * * * * /usr/bin/php -q /web/vhosts/cacti/poller.php > /var/log/poller.log 2>&1
	
	修改对应权限
	chown -R cactiuser.cactiuser cacti/log cacti/rra

	若是使用cmd.sh
	chmod 755 catcti/poller.sh cacti/cmd.php

	手动验证
	php  /web/vhosts/cacti/poller.php
	观察对应log和rra下rrd文件是否生成

spine替换cmd.php

	wget http://www.cacti.net/downloads/spine/cacti-spine-0.8.8c.tar.gz
	yum install net-snmp-devel mysql-devel
	./configure
	./make && make install

	cd /usr/local/spine/etc/
	mv spine.conf.dist /etc/spine.conf
	vi spine.conf
	DB_Host         localhost
	DB_Database     cactidb
	DB_User         cactiuser
	DB_Pass         cactiuser
	DB_Port         3306

	设置spine路径，Console——Settings——Paths
	cacti设置spine路径
	/usr/local/spine/bin/spine
	更改cacti轮询器为spine，Console——Settings——Poller
	cacti更改轮询器为spine

	运行：
	#/usr/local/spine/bin/spine
	SPINE: Using spine config file [/etc/spine.conf]
	SPINE: Version 0.8.7g starting
	SPINE: Time: 0.2410 s, Threads: 5, Hosts: 2
	
	说明：spine默认配置文件需要放在/etc才会生效，否则报如下错误：
	SPINE: Poller[0] FATAL: Unable to read configuration file! (Spine init)
		
###客户端设置###
	yum install net-snmp

	vim /etc/snmpd/snmpd.conf
	40 #       sec.name  source          community
 	41 com2sec notConfigUser  default       public
	61 #       group          context sec.model sec.level prefix read   write  notif
 	62 access  notConfigGroup ""      any       noauth    exact  all none none
	84 ##           incl/excl subtree                          mask
	85 view all    included  .1                               80
	
	service snmpd restart
	注意selinux,iptables，是否允许

批量工具推荐：[clustershell](./clustershell.md)

	clush -b -w @app yum -y install net-snmp
	clush -b -w @app -c /etc/snmp/snmpd.conf --dest=/etc/snmp
	clush -b -w @app chkconfig snmpd on
	clush -b -w @app service snmpd start
