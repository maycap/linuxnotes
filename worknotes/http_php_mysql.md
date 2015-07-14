##http\_php_mysql

###前言
常见的组合实例，以一款php应用为例


###步骤

>安装

	yum -y install httpd php mysql mysql-server php-mysql

>修改http
	
	1.修改Listen,ServerName
	2.若是启用.htaccess----修改为 AllowOverride All	

>配置mysql

	#启动服务
	service mysqld start

	#修改密码
	mysqladmin -u root password 'new-password'

	#修改权限
	mysql -u root -p
	mysql> DROP DATABASE test; 　　　　　　　　　　　　　　 [删除test数据库]
	mysql> DELETE FROM mysql.user WHERE user = ''; 　　　[删除匿名帐户]
	mysql> FLUSH PRIVILEGES; 　　　　　　　　　　　　　　　 [重载权限]

	#创建数据库，导入数据库,所有地址标示符为%
	mysql> CREATE DATABASE mytest;
	mysql> GRANT ALL PRIVILEGES ON mytest.* TO 'myuser'@'localhost' IDENTIFIED BY 'password';	
	
	mysql -u myuser -p mytest < test.sql

	#修改访问
	1.授权法：
		use mysql;
		grant all privileges  on *.* to leo@'%' identified by "leo";
		以leo用户在任何地方都可以访问；
	2.该表法：
		可以实现以root用户在任何地方访问数据库
		update user set host = '%' where user = 'root';
		这样就可以了

>php修改
	
	#关于/etc/php.ini中时区设置
	;date.timezone =

	#修改php配置用关于数据库信息的连接信息
			
>备用的扩展

	1.安装apache扩展
	yum -y install httpd-manual mod_ssl mod_perl mod_auth_mysql
	
	2.安装php的扩展
	yum install php-gd
	yum -y install php-gd php-xml php-mbstring php-ldap php-pear php-xmlrpc
	
	3.安装mysql扩展
	yum -y install mysql-connector-odbc mysql-devel libdbi-dbd-mysql
	
	
