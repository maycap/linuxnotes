
### vsftp匿名上传下载

内部使用，最快的简洁交换文件，匿名方式

1.0 修改主目录

	vim /etc/passwd
	ftp:x:14:50:FTP User:/web/ftp:/sbin/nologin

	chown -R root:ftp /web/ftp

1.1 匿名服务器的连接（独立的服务器）

	在/etc/vsftpd/vsftpd.conf配置文件中添加如下几项：
	anonymous_enable=yes (允许匿名登陆)
	dirmessage_enable=yes （切换目录时，显示目录下.message的内容）
	local_umask=022 (FTP上本地的文件权限，默认是077)
	connect_form_port_20=yes （启用FTP数据端口的数据连接）*
	xferlog_enable=yes （激活上传和下传的日志）
	xferlog_std_format=yes (使用标准的日志格式)
	ftpd_banner=XXXXX （欢迎信息）
	pam_service_name=vsftpd （验证方式）*
	listen=yes （独立的VSFTPD服务器）*
	功能：只能连接FTP服务器，不能上传和下传
	注：其中所有和日志欢迎信息相关连的都是可选项,打了星号的无论什么帐户都要添加，是属于FTP的基本选项
	
1.2 开启匿名FTP服务器上传权限

	在配置文件中添加以下的信息即可：
	anon_upload_enable=yes (开放上传权限)
	anon_mkdir_write_enable=yes （可创建目录的同时可以在此目录中上传文件）
	Write_enable=yes (开放本地用户写的权限)
	anon_other_write_enable=yes (匿名帐号可以有删除的权限)
	
1.3 开启匿名服务器下传的权限

	在配置文件中添加如下信息即可：
	anon_world_readable_only=no
	注：要注意文件夹的属性，匿名帐户是其它（other）用户要开启它的读写执行的权限
	（R）读-----下传 （W）写----上传 （X）执行----如果不开FTP的目录都进不去

处于安全，还是使用常规认证模式简单，配置也少

	chown -R  ftp:ftp /web/ftp
	usermod -d /web/ftp ftp

	anonymous_enable=NO
	注释匿名anon_*开头的即可

	chroot_local_user=YES  --限制为自身家目录，确保安全
	
### vsftp限速


>编辑/etc/vsftpd/vsftpd.conf

	 tcp_wrapper=YES
	 
>编辑/etc/hosts.allow

	vsftpd:192.168.*.*:setenv VSFTPD_LOAD_CONF /etc/vsftpd/local.class
	vsftpd:ALL:setenv VSFTPD_LOAD_CONF /etc/vsftpd/internet.class
	
>创建并编辑/etc/vsftpd/local.class

	#设置为0代表不限速
	anon_max_rate=0  #匿名登录用户
	local_max_rate=0    #授权登录用户
    
>创建并编辑/etc/vsftpd/internet.class
	
	anon_max_rate=300000  #匿名用户的速度约300KB/s
	local_max_rate=500000   #授权用户的速度约500KB/s
	
重启vsftpd服务，这样就实现对192.168.0.0这个网段不限速，其余网段限速的效果。


### vsftp ldap (AD) 认证

> /etc/vsftpd/vsftpd.conf 

	＃添加认证模块配置
	pam_service_name=vsftpd.ldap

> /etc/pam.d/vsftpd.ldap

	#%PAM-1.0
	auth      sufficient  pam_ldap.so
	account    sufficient  pam_ldap.so
	
> /etc/pam_ldap.conf

	host [ip]
	base OU=［code］,DC=［corp］,DC=cn
	binddn CN=[user],CN=Users,DC=［corp］,DC=cn
	bindpw  [password]
	pam_login_attribute sAMAccountName