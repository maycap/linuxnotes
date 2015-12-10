#RH7

***
####前言
去客户偶遇RH7，操作记录如下

***
####系统类修改
>修改主机名

	直接编辑 /etc/hostname，立即生效

>添加hosts解析
	vim /etc/hosts

	IP地址		  主机名  主机域
	172.24.17.x   Tbcsvc Tbcsvc.localdomain 

>配置centos-yum源

可用的替换rpm脚本，版本参数不对，自行查找替换即可

	#!/bin/sh

	#删除原有yum包
	rpm -aq|grep yum|xargs rpm -e --nodeps 
	
	#按照删除的查找对应的yum，依次下载
	wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-125.el7.centos.noarch.rpm
	wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
	wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-utils-1.1.31-29.el7.noarch.rpm
	wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-updateonboot-1.1.31-29.el7.noarch.rpm
	wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-29.el7.noarch.rpm
	
	#存在互相依赖，一次性导入
	rpm -ivh yum-*.rpm


添加repo，指向yum源，选择对应版本修改repo中的对应参数


	[root@]# cat /etc/yum.repos.d/CentOS-Base.repo 
	[base]
	name=CentOS-7 - Base
	baseurl=http://mirrors.163.com/centos/7/os/$basearch/
	gpgcheck=1
	gpgkey=http://mirrors.163.com/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7 	
	
	#released updates
	[updates]
	name=CentOS-7 - Updates
	baseurl=http://mirrors.163.com/centos/7/updates/$basearch/
	gpgcheck=1
	gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
	 
	#packages used/produced in the build but not released
	[addons]
	name=CentOS-7 - Addons
	baseurl=http://mirrors.163.com/centos/7/os/$basearch/
	#baseurl=http://mirrors.163.com/centos/7/addons/$basearch/
	gpgcheck=1
	#gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
	#additional packages that may be useful
	[extras]
	name=CentOS-7 - Extras
	baseurl=http://mirrors.163.com/centos/7/extras/$basearch/
	gpgcheck=1
	gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
	#additional packages that extend functionality of existing packages
	[centosplus]
	name=CentOS-7 - Plus
	baseurl=http://mirrors.163.com/centos/7/centosplus/$basearch/
	gpgcheck=1
	enabled=0

	
>关闭防火墙

	查看防火墙状态。
	
	#systemctl status firewalld
	
	临时关闭防火墙命令。重启电脑后，防火墙自动起来。
	
	systemctl stop firewalld
	
	永久关闭防火墙命令。重启后，防火墙不会自动启动。
	
	systemctl disable firewalld
	
	打开防火墙命令。
	
	systemctl enable firewalld