## bandwidthd

### 简介
简单的单机监控流量程序，安装配置都很方便


### 安装

>基本编译组件安装

	yum install gcc cpp glibc glibc-devel gcc-c++

>PCAP/PNG/GD Library（图像处理库）

	 yum install libpcap libpcap-devel libpng libpng-devel gd gd-devel

>前端服务器

	#httpd

	yum install httpd mod_ssl

	#nginx
	
>bandwidthd

	wget http://jaist.dl.sourceforge.net/project/bandwidthd/bandwidthd/bandwidthd%202.0.1/bandwidthd-2.0.1.tgz

	tar zxvf bandwidthd-2.0.1.tgz
	./configure
	make
	make install

### 配置

>目录介绍

	ll /usr/local/bandwidthd
	 
	-rwxr-xr-x 1 root root 53320 3月  19 15:15 bandwidthd  //启动bandwidthd文件
	 
	drwxr-xr-x 2 root root  4096 3月  19 15:51 etc          //配置文件
	 
	drwxr-xr-x 2 root root  4096 3月  19 15:25 htdocs      //web访问目录，可以作一个虚拟主机指过来
		
>配置修改

	vim  /usr/local/bandwidthd/etc/bandwidthd.conf

	subnet 10.1.3.0 255.255.255.0 #设置监控的网段
	
	dev "any" #(这是你要检测的网卡ethx或any(所有),可以调整为对应的网络连接设备)

	skip_intervals 1 #默认2.5 minutes 刷新

	graph_cutoff 1024 #默认1M 以上的流量才有图形

	#promiscuous true #设置网卡在混杂模式中记录

	output_cdf true #在bandwidthd目录中生成log2.cdf 以log.cdf格式数据记录

	recover_cdf true #在启动bandwidth时重新读取cdf的数据

	filter "ip" #以ip为过滤对象

	graph true #图形生成

	meta_refresh 150 #网页刷新时间

>网页展示

	#httpd

	ln -s /usr/local/bandwidthd/htdocs  /var/www/html/bandwidthd

	service httpd restart

	#nginx

	location /bandwidthd/ {
			alias /usr/local/bandwidthd/htdocs/;
		}

	nginx -s reload


>启动对应服务

	/usr/local/bandwidthd/bandwidthd

### 验收

	http://ip/bandwidthd/


	