##clustershell##
***
###前言###
轻量级linux运维利器，并发执行，统一配置文件

***
###安装clustershell


采用常规方式，需要添加epel源，参考 http://fedoraproject.org/wiki/EPEL 选取对应版本的‘epel-release’，添加源后即可查询到，以EL6为例：
	
	yum install http://mirrors.opencas.cn/epel/6/i386/epel-release-6-8.noarch.rpm
	yum -y install clustershell

clustershell是python编写，不支持windows，可用python方式安装，源码 https://github.com/cea-hpc/clustershell 

	pip install clustershell
	or
	git clone https://github.com/cea-hpc/clustershell
	python setup.py install

###简单使用###

nodeset可用于节点控制，简单点可以直接添加/etc/clustershell/groups

	#old vim /etc/clustershell/groups
	vim /etc/clustershell/groups.d/local.cfg
	hadoop: HSlave[1-3,6-17,20],HClient[1-2]

	需要添加ssh认证，hadoop集群默认已添加
	ssh-keygen -t rsa
	ssh-copy-id -i ~/.ssh/id_rsa.pub HSalve...

	简单测试可用
	clush -b -w  @hadoop hostname
	
	统一配置文件，本地为HMaster
	clush -b -w  @hadoop -c /web/hadoop-2.6.0/etc/hadoop/*-site.xml --dest=/web/hadoop-2.6.0/etc/hadoop

###详细使用###
参考wiki---https://github.com/cea-hpc/clustershell/wiki
	
