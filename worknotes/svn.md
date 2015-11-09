###SVN Synchronous apache###
***

###前言###

在未使用gitlab管理代码前，公司内部使用SVN，前端工程师为了代码及时展现效果，提出提交代码后，要在浏览器及时看到效果，如一些php文件等需要解析的前端文件。SVN异机钩子同步失败，决定本地更新，NFS挂载。


* ####同步代码

		mkdir webos
		svn co http://192.168.1.200/svn/repos/program/eln4/app/webos --username=  --password
		chown apache.apache -R webos

	由于服务器使用Apache承载SVN，其属主为apache,便于操作

* ####设置钩子

		vim /var/www/.subversion/servers
 		store-plaintext-passwords = no
		#svn缓冲明文密码默认需要恢复yes或者no确认，去掉服务器确认

		cp /root/.subversion /var/www/
		chown -R apache.apache /var/www
		#确保权限一直，以为服务属主是Apache，执行钩子的目录必须是其拥有可执行权

		cp hooks/post-commit.tmpl  hooks/post-commit
		chmod +x post-commit

		cat post-commit
		1. #!/bin/sh  
		2. REPOS="$1"  
		3. REV="$2"  
		4. export LANG=en_US.UTF-8  
		5. /usr/bin/svn update /web/php_sys_svn --username user --password xxxxxx 2>>/tmp/svn_hook_log.txt  
		6. echo `whoami`,$REPOS,$REV >> /tmp/svn_hook_var.txt  

	鉴于字符集编码已经祸害了无数人，请在脚本中加入编码

* ####NFS挂载

		cat /etc/exports
		/web/php_sys_svn *(rw,async,no_root_squash)

	首先要确信你已经安装nfs，并启动了。

* ####Apache展示服务器
		mount -t nfs 192.168.1.200:/web/php_sys_svn /web/php_sys_svn
		ln -s /web/php_sys_svn **(apache 家目录）
	网页访问确认即可。


###备份###

>环境

	主机：192.168.1.200
	从机：192.168.1.201

>备份机前期操作

	mkdir -p /web/svnbak/repos
	svnadmin create /web/svnbak/repos
	
	cd /web/svnbak/repos/hooks
	cp pre-revprop-change.tmpl pre-revprop-change
	chmod 755 pre-revprop-change

	#修改pre-revprop-change内容
	echo “Changing revision properties other than svn:log is prohibited” >&2
	exit 0（1修改为0）
	
> 初始化

	svnsync init file:///web/svnbak/repos http://192.168.1.200/svn/repos --username test --password test
	
	会出现以下信息：
	Copied properties for revision 0.

>同步文件

	svnsync sync file:///web/svnbak/repos

>删lock

	svn propdel svn:sync-lock --revprop -r 0 file:///web/svnbak/repos

>httpd -- subversion.conf

	#yum install httpd subversion mod_dav_svn

	<Location /svn/ >
		DAV svn
		SVNParentPath /web/svnbak
		SVNListParentPath on
		AuthType Basic
		AuthName "Authorization Realm"
		AuthUserFile /web/svnbak/repos/conf/passwd
		AuthzSVNAccessFile  /web/svnbak/repos/conf/authz
		Require valid-user
	</Location>

建议使用 /svn/ 作为基目录，repos为父目录，惨不忍睹

>主动钩子同步

为了同时同步，想要在主服务器上使用钩子，备份服务器必须可以远程访问。采用传统的apache模式，很容易做到。切记账户分清，只有同步账户可写，其余都是只读。

	#前提 先手动执行下，存下主机密码

	vim post-commit
	usr/bin/svnsync sync http://192.168.100.14/svn/repos --username=user  --password=passwd  2>>/tmp/svnsync.log



>bug

第二天过来磁盘满了，原始svn才3.8G，svnbak占用了173G,重新sync 报错如下

	svnsync: OPTIONS of 'http://192.168.1.200/svn/repos': 200 OK 
解释为不存在，修改源httpd--- /svn/  -- ，重新开始，报错为

	svn: E130003: The OPTIONS response contains invalid XML (200 OK)	
查看源服务器，得知原始路径为8000，使用的地址为转发地址，可能导致回环，重新测试

	mkdir /opt/svn
	svnadmin create /opt/svn/repos
	svnsync init file:///opt/svn/repos/ http://192.168.1.200:8000/svn/repos
	svnsync sync file:///opt/svn/repos/

问题得以解决

>bug2

手动执行可以同步，钩子调用失败没报错如下：

	svnsync: /home/apache/.subversion/servers:1: Section header expected
	
	-----------------------------------------------------------------------
	ATTENTION!  Your password for authentication realm:

原因为：自动调用需要验证密码缓存，导致失败，添加 --no-auth-cache

	vim post-commit
	usr/bin/svnsync sync http://192.168.100.14/svn/repos --username=user  --password=passwd  2>>/tmp/svnsync.log  --no-auth-cache


***
###源码安装最新版###
yum虽然方便，版本总是落后，而且莫名的bug也不好理解，先更新到最新版试试。官网下载地址 http://subversion.apache.org/download/ ，下载不了，请切换 Mirror。新版依赖serf,官网地址https://code.google.com/p/serf/

	wget http://apache.dataguru.cn/subversion/subversion-1.8.13.tar.bz2
	sha1sum subversion-1.8.13.tar.bz2   ---肉眼看下，对比下

	https://code.google.com/p/serf/
	下载丢进 subversion-1.8.13/temp 即可

	cd  subversion-1.8.13
	./get-deps.sh   --安装依赖
	./configure --prefix=/usr/local/subversion --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr --enable-mod-activation --with-apache-libexecdir=/usr/local/httpd/modules --with-apxs=/usr/sbin/apxs --with-serf=/usr/local/serf
	./make && make install

###单独编译mod\_dav\_svn###
>需要安装httpd-devel

	yum install httpd-devel

>解压编译

	./configure --with-apxs=/usr/sbin/apxs
	make && make install

>替换

	到/usr/local/libexec，将mod_svn_dav.so和mod_authz_svn.so复制到/etc/httpd/modules/下。
	确认apache配置文件中有如下两行：
	LoadModule dav_svn_module modules/mod_dav_svn.so
	LoadModule authz_svn_module modules/mod_authz_svn.so
