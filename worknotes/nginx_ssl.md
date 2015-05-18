###nginx_ssl###
***

###前言
公司使用nginx作为web服务器，一些功能需要加密，在内部开发时自定义证书

***

###nginx主配置
	server {
	        listen       443;
	        server_name  www.jaylin.org;
	        ssl on;
	        ssl_certificate /etc/nginx/jaylin.crt;
	        ssl_certificate_key /etc/nginx/jaylin.key;
			ssl_session_timeout  5m;
	        ssl_protocols  SSLv3 TLSv1;
	        ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
	        ssl_prefer_server_ciphers   on;
	        ......
	}
	由于功能相互依赖，被依赖的应用，location /myapp/ 也要加入ssl配置中，具体可通过https访问chrome查看报错提示添加。

###自定义证书

	# cd /etc/nginx
	# openssl genrsa -des3 -out jaylin.key 1024
	# openssl req -new -days 3650 -key jaylin.key -out jaylin.csr
	# cp jaylin.key jaylin.key.orgi
	# openssl rsa -in jaylin.key.orgi -out jaylin.key
	# openssl x509 -req -days 365 -in jaylin.csr -signkey jaylin.key -out jaylin.crt

	采用简单的方式可以生成密钥对，却避免不了重启nginx需要手动输入必须设置的密码，不利于自动化启动与管理。