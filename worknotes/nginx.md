###nginx###
***

###前言
公司使用nginx作为web服务器，一些功能需要加密，在内部开发时自定义证书。和nginx一些遇到问题及解决方法。

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
	由于功能相互依赖，被依赖的应用，location /myapp/ 也要加入ssl配置中。
	具体可通过https访问应用，chrome查看报错提示，依次添加。

###自定义证书

	# cd /etc/nginx
	# openssl genrsa -des3 -out jaylin.key 1024
	# openssl req -new -key jaylin.key -out jaylin.csr
	# cp jaylin.key jaylin.key.orgi
	# openssl rsa -in jaylin.key.orgi -out jaylin.key
	# openssl x509 -req -days 3650 -in jaylin.csr -signkey jaylin.key -out jaylin.crt

	采用简单的方式可以生成密钥对，却避免不了重启nginx需要手动输入必须设置的密码，不利于自动化启动与管理。

***
###代理websocket

在使用nginx代理websocket时，使用最简单配置如下：

	  location / {
                proxy_pass http://10.117.27.139;
                proxy_set_header Host $host;
		}

调用websocket报错如下：
	
	javax.websocket.DeploymentException: The HTTP response from the server [HTTP/1.1 404 Not Found

修改配置选项：
	
  	proxy_pass http://10.117.27.139:80;
    proxy_set_header Host $host;

	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_http_version 1.1;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";
