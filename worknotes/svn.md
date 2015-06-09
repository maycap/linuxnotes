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