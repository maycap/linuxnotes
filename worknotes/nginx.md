###nginx###
***

###前言
公司使用nginx作为web服务器，一些功能需要加密，在内部开发时自定义证书。和nginx一些遇到问题及解决方法。

***

###nginx升级

>源码下载

	http://nginx.org/download/ 

>获取配置、编译

	nginx -V 
	
	./configure  (configure arguments)
	make && make install

>指定独立用户，或者使用root
	
	groupadd nginx
	useradd -M -s /sbin/nologin -g nginx nginx
	chown -R nginx.nginx /web/nginx

>TROUBSHOOT

	#before configure nginx
   	yum install gcc openssl-devel pcre-devel zlib-devel

	#libGeoIP.so.1 error
	ln -s /usr/local/lib/libGeoIP.so.1 /usr/lib64/

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

###静态文件缓存

	#缓存目录设置
	proxy_temp_path   /web/nginx/proxy_temp_dir 1 2;
	
	proxy_cache_path  /usr/local/nginx/proxy_cache_dir/cache1  levels=1:2 keys_zone=cache1:100m inactive=1d max_size=10g;

	#keys_zone=cache1:100m 表示这个zone名称为cache1，分配的内存大小为100MB
	#/usr/local/nginx/proxy_cache_dir/cache1 表示cache1这个zone的文件要存放的目录
	#levels=1:2 表示缓存目录的第一级目录是1个字符，第二级目录是2个字符，即/usr/local/nginx/proxy_cache_dir/cache1/a/1b这种形式
	#inactive=1d 表示这个zone中的缓存文件如果在1天内都没有被访问，那么文件会被cache manager进程删除掉
	#max_size=10g 表示这个zone的硬盘容量为10GB


	#在日志格式中加入$upstream_cache_status

	#具体location设置
	#设置资源缓存的zone

    proxy_cache cache1;   --缓存名称

    #设置缓存的key
    proxy_cache_key $host$uri$is_args$args;

    #设置状态码为200和304的响应可以进行缓存，并且缓存时间为10分钟
    proxy_cache_valid 200 304 10m;

	#添加回应头中的缓存命中状况(MISS,HIT...）
	add_header X-Cache '$upstream_cache_status';

    expires 30d;

	-----------------备注----------------------
	alias /web/eln4share/web-static;
	这种文件就在本地，是不会走缓存的

	proxy_set_header X-FORWARDED-FOR $proxy_add_x_forwarded_for;
	proxy_set_header Host $host;
	proxy_pass http://xxx.xxx.com/web-static/;
	
	这种才会先缓存到 proxy_temp,再写入proxy_path。因此在proxy_path中可以看到缓存文件

	
***

###下载静态文件出现错误

>ERR\_CONTENT\_LENGTH\_MISMATCH

某个应用调用abc.js出现如下错误：错误 354 (net::ERR_CONTENT_LENGTH_MISMATCH)：服务器意外关闭了连接。单独访问js所在路径，发现是可以的。查看nginx错误日志：

	tail -f logs/error.log

	2015/08/04 11:04:00 [crit] 15320#0: *6667 open() "/web/nginx/proxy_temp/0/44/0000001440" failed (13: Permission denied) while reading upstream

原因：nginx会缓存大文件到proxy_temp目录中，然而对这个目录没有读写权限

查看对应proxy_temp 权限发现是原同事copy过来，权限为nginx。而nginx配置为并未设置启动用户，采用了默认了nobody账户，导致proxy_temp没有权限写入。

	#添加一项配置或者去掉注释：
	user  nginx;
 
	#reload
	/web/nginx/sbin/nginx -t
	/web/nginx/sbin/nginx -s reload

