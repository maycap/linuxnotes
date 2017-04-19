## TroubleShoot

***

### 前言
公司内部遇到的一些环境小点，零零散散，记录备用

***
### javasqlite
常规流程：

	步骤 1:yum install sqlite3*  sqlite-devel
	步骤 2:tar -xvf javasqlite-20130214.tar.gz
	步骤 3:./configure --prefix=/opt/sqlite
	步骤 4:make && make install
	步骤 5:cp /opt/sqlite/lib/* /sbin/lib64/ -a
		/bin/cp /opt/sqlite/lib/* /usr/lib/ -a
	步骤6：如上述5失效，复制到jdk调用目录下

>./configure 报错找不到 try --with-sqlite/--with-sqlite3

	strace 追踪得知找不到sqlite.h
	yum install sqlite-devel

>复制步骤五后，代码提示

	java.lang.UnsatisfiedLinkError: SQLite.Database.internal_init()V
	or
	Could not initialize class SQLite.Database

	jdk调用不到libsqlite_jni.la  libsqlite_jni.so
	这两个文件就是编译后在/opt/sqlite/lib的文件，
	重点是要让程序可以调用，或者在系统环境中，或者在CLASSPATH指定的目录下。
	echo $CLASSPATH ，复制过去
	or
	vi /etc/profile 添加CLASSPATH

***
### jdk字体
为了生成一些人性化的证书，需要引入windows的一些字体进入jdk，而不是linux的系统环境字体。

	获取字体:
	从本地 C:\Windows\Fonts 或者网络或许想要的字体文件（*.ttf文件）--> fallback
	
	上传到服务器修改权限:
	chmod -R 755 fallback

	mkfontscale  (command not found # yum install mkfontscale)
	mkfontdir 
	
	restart tomcat 

由于linux是文件系统，可以直接复制使用，另外一台服务器若要使用，直接scp过去即可。


### jdk1.7+验证码问题

>报错提示参考

	Could not initialize class sun.awt.X11FontManager

	/libfontmanager.so: libgcc_s.so.1: cannot open shared object file: No such file or directory

>对应安装

	#yum install dejavu*
	yum groupinstall Fonts
	yum install libgcc_s.so.1
	ldconfig

	restart tomcat	


***
### 执行sh编码问题

	: No such file or direct﻿orybin/sh
	: command not founde 2: 

	vim 
	set ff=unix

	line 1: ﻿#!/bin/bash: No such file or directory

	touch test.sh	
	cat XXXX  
	除去第一行，重新复制到新建脚本中即可

***

### vsftp匿名上传下载

内部使用，最快的简洁交换文件，匿名方式

1.0 修改主目录

	vim /etc/passwd
	ftp:x:14:50:FTP User:/web/ftp:/sbin/nologin

	chown -R root:ftp /web/ftp

1.1 匿名服务器的连接（独立的服务器）

	在/etc/vsftpd/vsftpd.conf配置文件中添加如下几项：
	anonymous_enable=yes (允许匿名登陆)
	dirmessage_enable=yes （切换目录时，显示目录下.message的内容）
	local_umask=022 (FTP上本地的文件权限，默认是077)
	connect_form_port_20=yes （启用FTP数据端口的数据连接）*
	xferlog_enable=yes （激活上传和下传的日志）
	xferlog_std_format=yes (使用标准的日志格式)
	ftpd_banner=XXXXX （欢迎信息）
	pam_service_name=vsftpd （验证方式）*
	listen=yes （独立的VSFTPD服务器）*
	功能：只能连接FTP服务器，不能上传和下传
	注：其中所有和日志欢迎信息相关连的都是可选项,打了星号的无论什么帐户都要添加，是属于FTP的基本选项
	
1.2 开启匿名FTP服务器上传权限

	在配置文件中添加以下的信息即可：
	anon_upload_enable=yes (开放上传权限)
	anon_mkdir_write_enable=yes （可创建目录的同时可以在此目录中上传文件）
	Write_enable=yes (开放本地用户写的权限)
	anon_other_write_enable=yes (匿名帐号可以有删除的权限)
	
1.3 开启匿名服务器下传的权限

	在配置文件中添加如下信息即可：
	anon_world_readable_only=no
	注：要注意文件夹的属性，匿名帐户是其它（other）用户要开启它的读写执行的权限
	（R）读-----下传 （W）写----上传 （X）执行----如果不开FTP的目录都进不去

处于安全，还是使用常规认证模式简单，配置也少

	chown -R  ftp:ftp /web/ftp
	usermod -d /web/ftp ftp

	anonymous_enable=NO
	注释匿名anon_*开头的即可

	chroot_local_user=YES  --限制为自身家目录，确保安全
	
### vsftp限速


>编辑/etc/vsftpd/vsftpd.conf

	 tcp_wrapper=YES
	 
>编辑/etc/hosts.allow

	vsftpd:192.168.*.*:setenv VSFTPD_LOAD_CONF /etc/vsftpd/local.class
	vsftpd:ALL:setenv VSFTPD_LOAD_CONF /etc/vsftpd/internet.class
	
>创建并编辑/etc/vsftpd/local.class

	#设置为0代表不限速
	anon_max_rate=0  #匿名登录用户
	local_max_rate=0    #授权登录用户
    
>创建并编辑/etc/vsftpd/internet.class
	
	anon_max_rate=300000  #匿名用户的速度约300KB/s
	local_max_rate=500000   #授权用户的速度约500KB/s
	
重启vsftpd服务，这样就实现对192.168.0.0这个网段不限速，其余网段限速的效果。


### 编译
在使用基础内核时，一般没有gcc，g++，编译源码前需要
	
	yum -y install gcc gcc-c++


### 导出失败

导出列表，小文件可以，大文件有的浏览器导出丢失，有的直接报错。tomcat日志如下：

	 Broken pipe

Broken pipe产生的原因通常是当管道读端没有在读，而管道的写端继续有线程在写，就会造成管道中断。（由于管道是单向通信的） SIGSEGV(Segment fault)意味着指针所对应的地址是无效地址，没有物理内存对应该地址。折腾许久，查看nginx日志发现，为proxy_temp没权限，修改结束。