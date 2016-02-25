##SUSE简单手册

###简介
基于suse11，操作环境vmware虚拟机


###系统类管理

>防火墙

	# chkconfig --list | grep fire
	SuSEfirewall2_init 0:off 1:off 2:off 3:off 4:off 5:off 6:off B:on
	SuSEfirewall2_setup 0:off 1:off 2:off 3:off 4:off 5:off 6:off
	可以看到B是on的状态，下面的命令来进行关闭B.
	# chkconfig --level B SuSEfirewall2_init off
	 
	或:
	# chkconfig --list | grep fire
	SuSEfirewall2_init 0:off 1:off 2:off 3:off 4:off 5:off 6:off B:on
	SuSEfirewall2_setup 0:off 1:off 2:off 3:on 4:off 5:on 6:off
	# chkconfig --level 3 SuSEfirewall2_setup off
	# chkconfig --level 5 SuSEfirewall2_setup off
	
	/etc/init.d/SuSEfirewall2_setup  stop
	/etc/init.d/SuSEfirewall2_init  stop

>默认语言

	1. 修改 /etc/sysconfig/language
	将文档中 RC_LANG 配置为 "en_US.UTF-8"
	保存并退出

	2. 运行SuSEconfig

	-----下述生效----

	vim /etc/profile

	export LC_ALL="en_US.UTF-8"
	export LANG="en_US.UTF-8"

>开机自启动设置

	#不同于Centos的/etc/rc.local,参考学习
	more /etc/rc.d/README   --系统启动说明指南

	At  boot  time, the boot level master script /etc/init.d/boot is called
    to initialise the system (e.g. file system check, ...).  It  also  exe-
    cutes some hardware init scripts linked into /etc/init.d/boot.d/.  Then
    it calls /etc/init.d/boot.local, which executes the local commands.

	对应文件为 /etc/init.d/boot.local

	boot.local中命令启动完毕，才会启动ip配置。因此若命令中涉及ip访问之类，将开不机！


###编译类

>configure异常

	checking whether the C compiler works... no
	...
	configure: error: C compiler cannot create executables

	
	-------
	zypper install gcc glibc-devel gcc-c++*


	configure: error: C++ preprocessor "/lib/cpp" fails sanity check

	
	-------
	glibc 2.10以上比较稳定，低于缺少一些常见依赖包，导致
	SUSE 11.0 默认gcc 为2.9,缺少一些依赖
	SUSE 11 SP2 ，gcc 为2.11，编译顺利

>一些依赖包rpm建议
	
	#基于nginx编译
	rpm -ivh zlib-devel-1.2.3-106.34.x86_64.rpm

	#11SP2的openssl包为0.9.8j,选择对应版本的rpm
	rpm -ivh libopenssl-devel-0.9.8j-0.26.1.x86_64.rpm 

	#基于postgresql编译
	#数据库编译同样依赖zlib-devel，已导入不必重复
	rpm -ivh readline-devel-5.2-147.3.x86_64.rpm 

	