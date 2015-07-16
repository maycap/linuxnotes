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


###代理mail
一般情况下，客户端发起的邮件请求在经过nginx这个邮件代理服务器后，从网络通信的角度来看，nginx实现邮件代理功能时会把一个请求分为以下4个阶段：

	接收并解析客户端初始请求的阶段。
	向认证服务器验证请求合法性，并获取上游邮件服务器地址的阶段。
	nginx根据用户信息多次与上游服务器交互验证合法性的阶段。
	nginx在客户端与上游邮件服务器间纯粹透传TCP流的阶段。

一般情况下，客户端发起的邮件请求在经过nginx这个邮件代理服务器后，从网络通信的角度来看，nginx实现邮件代理功能时会把一个请求分为以下4个阶段：

	mail {  
	    // 邮件认证服务器的访问URL  
	    auth_http IP:PORT/auth.php;  
	  
	    // 当透传上，下游间的TCP流时，每个请求所使用的内存缓冲区大小  
	    proxy_buffer 4k;  
	  
	    server {  
	        /*对于POP3协议，通常都是监听110端口。POP3协议接收初始客户端请求的缓冲区固定为128字节，配置文件中无法设置*/  
	        listen 110;  
	        protocol pop3;  
	        proxy on;  
	    }  
	  
	    server {  
	        // 对于IMAP，通常都是监听143端口  
	        listen 143;  
	        protocol imap;  
	        // 设置接收初始客户端请求的缓冲区大小  
	        imap_client_buffer 4k;  
	        proxy on;  
	    }  
	  
	    server {  
	        // 对于SMTP，通常都是监听25端口  
	        listen 25;  
	        protocol smtp;  
	        proxy on;  
	        // 设置接收初始客户端请求的缓冲区大小  
	        smtp_client_buffer 4k;  
	    }  
	}

问题：

	nginx: [emerg] unknown directive "mail" in 
	
	请检查nginx -V  是否含有  --with-mail
	

###密码访问控制

有时候部署升级程序，需要外网访问入口，通过nginx设置密码，达到简单控制目的。

	#nginx简单配置
	location /upgrade/ { 
			proxy_pass http://cloud4:7070;
            auth_basic "secret";  //虚拟主机认证命名 
            auth_basic_user_file /usr/local/nginx/passwd.db; //虚拟主机用户名密码认证数据库 
        }

	#用htpasswd生成密码,可以在其他机器生成，然后传输过去
	htpasswd -c htpasswd.db upgrade

	chmod 400 htpasswd.db   //相当更安全些  

	

	 