##postfix

###前言
记录搭建本地邮件服务器遇到的一些问题

###问题记录

>本地邮件发送失败

	vim /etc/dovecot/dovecoto.conf

	#添加协议
	protocols = lmtp  imap  pop3
	

	vim /etc/dovecot/conf.d/10-master.conf

	#启动服务监听socket
	service lmtp {
	    unix_listener /var/spool/postfix/private/dovecot-lmtp {
	    mode = 0600
	    user = postfix
	    group = postfix
	  }
	}


	vim /etc/postfix/main.cf

	#本地邮件传输定义
	mailbox_transport = lmtp:unix:private/dovecot-lmtp
	virtual_transport = lmtp:unix:private/dovecot-lmtp

>网络邮件发送失败

	1、先做好dns解析

	2、postfix使用自定义的目录，在/var/spool/postfix/下
	cat /var/spool/postfix/etc/resolv.conf
	nameserver 192.168.1.1

>邮件过滤

	在/etc/postfix/main.cf中，启动 smtpd_recipient_restrictions，设置对应访问表

	check_recipient_access		匹配到的域名邮件丢弃，不发送
	check_client_access 		通过客户端ip，ip段或主机名屏蔽
	check_sender_access			通过判断发件人邮件地址（位于from段）屏蔽

	#添加方式
	smtpd_recipient_restrictions =
       check_recipient_access hash:/etc/postfix/recipient_access,
	   check_client_access hash:/etc/postfix/client_checks,
	   check_sender_access hash:/etc/postfix/sender_checks,
	   ....

	#参考配置
	# cat /etc/postfix/client_checks
	example.com               REJECT No spammers
	.example.com              REJECT No spammers, from your subdomain
	123.456.789.123           REJECT Your IP is spammer
	123.456.789.0/24          REJECT Your IP range is spammer
	321.987.654.321           OK
	example1.com              OK
	
	# cat /etc/postfix/sender_checks
	example.com              REJECT env. from addr any@example.com rejected
	.example.com             REJECT env. from addr any@sub.example.com rejected
	user@example.com         REJECT We don't want your email
	example2.com             OK
		
	#cat /etc/postfix/recipient_access
	xxx@example.com   		REJECT  #具体地址
	example.com             REJECT  #对整个域实现访问控制

	