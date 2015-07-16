##squid

###前言
squid是款优秀的代理服务器,普通代理、透明代理、反向代理。作为服务器加速使用，采用反向代理，缓冲静态文件，达到加速目的。


###源码安装###

>下载地址，选择最新稳定版

	http://www.squid-cache.org/Versions/

>安装流程，基于最少安装

	yum -y install gcc gcc-c++
	wget http://www.squid-cache.org/Versions/v3/3.5/squid-3.5.5.tar.gz
	tar zxvf squid-3.5.5.tar.gz
	cd squid-3.5.5
	./configure --prefix=/usr/local/squid
	make && make install

>配置初始，参考手册sqduid.conf.documented

	http_port 80 accel vhost 
	http_access allow all 
	cache_peer 192.168.100.82 parent 80 0 originserver round-robin weight=1 
	cache_peer 192.168.100.83 parent 80 0 originserver round-robin weight=1 
	visible_hostname my.test.com 
	cache_mgr gencat@163.com
	
	
	http_port 3128
	cache_mem 128 MB
	maximum_object_size 4 MB
	minimum_object_size 0 KB
	maximum_object_size_in_memory	 4096 KB
	cache_dir ufs /usr/local/squid/var/cache/squid 100 16 256
	access_log daemon:/usr/local/squid/var/logs/access.log squid
	cache_log /usr/local/squid/var/logs/cache.log
	
	logformat combined   %>a %[ui %[un [%tl] "%rm %ru HTTP/%rv" %>Hs %<st "%{Referer}>h" "%{User-Agent}>h" %Ss:%Sh
	
	#当cache高于95%的时候开始删除cache,直到保持80%的cache容量
	cache_swap_low 80
	cache_swap_high 95	

	#允许使用purge清空缓存
	acl AdminBoxes src 127.0.0.1 172.16.0.1 192.168.0.1
	acl Purge method PURGE
	http_access allow AdminBoxes Purge
	http_access deny Purge

	cache_effective_user nobody	
	cache_effective_group nobody	

###错误处理


1. 权限问题，squid默认用户为nobody

	/usr/local/squid/var/logs/cache.log: Permission denied

	see: (in squid.conf)
	
		cache_effective_user   nobody
		cache_effective_group  nobody

	chown -R nobody:nobody /usr/local/squid/var


###参考清空脚本

>cat clear_squid.cache.sh 

	#!/bin/sh

	#缓存路径
	squidcache_path=/usr/local/squid/var/cache/squid
	squidclient_path=/usr/local/squid/bin/squidclient

	grep -a -r $1 $squidcache_path/* | strings | grep "http:"| awk -F'ttp:' '{print "http:" $2;}' > cache_list.list
	for url in `cat cache_list.list`; do
		$squidclient_path -m PURGE -p 80 $url
	done
	
>用法：

	1、清除所有Flash缓存（扩展名.swf）：
	./clear_squid_cache.sh swf
	2、清除URL中包含sina.com.cn的所有缓存：
	./clear_squid_cache.sh sina.com.cn
	3、清除文件名为zhangyan.jpg的所有缓存：
	./clear_squid_cache.sh zhangyan.jpg



	




