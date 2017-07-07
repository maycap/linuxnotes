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


### 编译
在使用基础内核时，一般没有gcc，g++，编译源码前需要
	
	yum -y install gcc gcc-c++


### 导出失败

导出列表，小文件可以，大文件有的浏览器导出丢失，有的直接报错。tomcat日志如下：

	 Broken pipe

Broken pipe产生的原因通常是当管道读端没有在读，而管道的写端继续有线程在写，就会造成管道中断。（由于管道是单向通信的） SIGSEGV(Segment fault)意味着指针所对应的地址是无效地址，没有物理内存对应该地址。折腾许久，查看nginx日志发现，为proxy_temp没权限，修改结束。