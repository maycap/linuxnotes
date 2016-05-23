#apache_httpd

***
记录关于apache--httpd的一些配置。


####httpd转发监听

	ProxyPass /nexus http://127.0.0.1:8081/nexus

#####虚拟域名配置

	<VirtualHost *:80>
	    ServerName mail.test.com
	    ServerAlias mail.test.com
	    DocumentRoot "/var/www/html/webmail"
	    <Directory "/var/www/html/webmail">
	        Options Indexes FollowSymLinks
	        AllowOverride all
	        Order Allow,Deny
	        Allow from all
	    </Directory>
	</VirtualHost>

