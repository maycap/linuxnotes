##python
***

###前言###
centos6.5默认是python2.6.6，支持一些内容，需要升级


###获取源码

python官网下载地址是：https://www.python.org/downloads，参考实例：

	yum install -y zlib zlib-devel openssl   openssl-devel
	#来个倒叙，否则惨不忍睹，又要重新编译--pip

	yum install -y readline readline-devel sqlite3
	#安装ipython需要的插件
	
	
	wget https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tar.xz
	tar xvf Python-2.7.9.tar.xz
	cd Python-2.7.9
	make all
	make install
	make clean
	make distclean

###配置修改
安装python2.7.9后，默认仍是指向python2.6，需要重新指向。由于yum依赖python2.6，需要保留原有的python2.6。
	
	python -V
	which python
		/usr/local/bin/python
	#/usr/bin/python and /usr/bin/python2.6 是同一个文件
	mv /usr/bin/python /usr/bin/python2.6.6	
	ln -s /usr/local/bin/python2.7 /usr/bin/python

	#重新检验
	python -V


###修复原有连接
>yum

	which yum
	vim /usr/bin/yum
	配置文件头部	
	#!/usr/bin/python2.6.6
	
>iBus 输入法相关

	vi /usr/bin/ibus-setup  
	vi /usr/libexec/ibus-ui-gtk  
	

###安装pip

	wget https://bootstrap.pypa.io/get-pip.py --no-check-certificate
	 

troubleshoot:

	RuntimeError: Compression requires the (missing) zlib module
	yum install zlib zlib-devel
	#然后重新编译python2.7，否则将出现python2.7仍找不到zlib

	ImportError: No module named 'pip._vendor.requests'
	yum install -y openssl   openssl-devel
	#然后重新编译python2.7
	
源码安装：	

	wget https://pypi.python.org/packages/source/s/setuptools/setuptools-16.0.tar.gz --no-check-certificate
	tar zxvf setuptools-16.0.tar.gz
	python setup.py install

	wget https://pypi.python.org/packages/source/p/pip/pip-6.1.1.tar.gz --no-check-certificate
	tar zxvf pip-6.1.1.tar.gz
	python setup.py install

###安装ipython
为了调试方便，ipython是个不错的测试工具，若是提示warnning 警告，需要yum install，然后重新编译python2.7.9。
	
	pip install ipython



###salt
	#安装pyzmq报错，提示安装
	gcc: error trying to exec 'cc1plus': execvp
	yum install gcc-c++

	ImportError: No module named zmq
	pip install pyzmq
	

	