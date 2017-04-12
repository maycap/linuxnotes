## mysql

### 安装

>rpm方式

	#删除老的，才能换新的
	yum erase mysql-*
	
	#安装即可
	rpm -ivh MySQL-server-5.6.29-1.el6.x86_64.rpm 
  	rpm -ivh MySQL-devel-5.6.29-1.el6.x86_64.rpm 
  	rpm -ivh MySQL-client-5.6.29-1.el6.x86_64.rpm 


#### 初始化数据库

	#指定用户初始化

	mkdir -p /web/data_3306/data

	mysql_install_db --user=mysql --datadir=/web/data_3306/data/

	
#### 修改配置文件

	
	cat /web/data_3306/my.cnf 


	# For advice on how to change settings please see
	# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html
	
	[client]
	character-set-server = utf8
	port    = 3306
	socket  = /tmp/mysql_3306.sock
	
	
	
	[mysqld]
	
	# Remove leading # and set to the amount of RAM for the most important data
	# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
	# innodb_buffer_pool_size = 128M
	
	# Remove leading # to turn on a very important data integrity option: logging
	# changes to the binary log between backups.
	# log_bin
	
	# These are commonly set, remove the # and set as required.
	 basedir = /usr/local/mysql
	 datadir = /web/data_3306/data
	 port = 3306
	# server_id = .....
	 socket = /tmp/mysql_3306.sock
	language = /usr/share/mysql/english
	
	log-error = /web/data_3306/mysql_error.log
	pid-file = /web/data_3306/mysql.pid
	
	# Remove leading # to set options mainly useful for reporting servers.
	# The server defaults are faster for transactions and fast SELECTs.
	# Adjust sizes as needed, experiment to find the optimal values.
	# join_buffer_size = 128M
	# sort_buffer_size = 2M
	# read_rnd_buffer_size = 2M 
	
	sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 


#### 修改初始密码

	#跳过授权，允许本地登录
	mysqld_safe --defaults-file=/web/data_3306/my.cnf --skip-grant-tables --skip-networking

	#通过socket登录
	mysql -S /tmp/mysql_3306.sock -u root -p

	mysql>use mysql
	mysql> UPDATE user SET Password=PASSWORD('newpassword') where USER='root' and host='root' or host='localhost';
	mysql>FLUSH PRIVILEGES;
	mysql>quit




***


### troubleshoot

>[ERROR] Can't find messagefile '/usr/share/mysql/errmsg.sys'

	配置文件指定语言目录
	language = /usr/share/mysql/english

>connect to server at 'localhost' failed

	启动时添加选项
	--skip-networking