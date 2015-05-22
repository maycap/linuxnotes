##TroubleShoot##

***

###前言###
公司内部遇到的一些环境小点，零零散散，记录备用

***
###javasqlite###
常规流程：

	步骤 1:yum install sqlite3*  sqlite-devel
	步骤 2:tar -xvf javasqlite-20130214.tar.gz
	步骤 3:./configure --prefix=/opt/sqlite
	步骤 4:make && make install
	步骤 5:cp /opt/sqlite/lib/* /sbin/lib64/

>./configure 报错找不到 try --with-sqlite/--with-sqlite3

	strace 追踪得知找不到sqlite.h
	yum install sqlite-devel

>复制步骤五后，代码提示

	java.lang.UnsatisfiedLinkError: SQLite.Database.internal_init()V
	or
	Could not initialize class SQLite.Database

	jdk调用不到libsqlite_jni.la  libsqlite_jni.so
	这两个文件就是编译后在/opt/sqlite/lib的文件，其实复制不是重点。
	重点是要让jdk可以调用，也就是在其CLASSPATH指定的目录下。
	echo $CLASSPATH ，复制过去
	or
	vi /etc/profile 添加CLASSPATH

***
###jdk字体###
为了生成一些人性化的证书，需要引入windows的一些字体进入jdk，而不是linux的系统环境字体。

	获取字体:
	从本地 C:\Windows\Fonts 或者网络或许想要的字体文件（*.ttf文件）--> fallback
	
	上传到服务器修改权限:
	chmod -R 755 fallback

	mkfontscale  (command not found # yum install mkfontscale)
	mkfontdir 
	
	restart tomcat 

由于linux是文件系统，可以直接复制使用，另外一台服务器若要使用，直接scp过去即可。