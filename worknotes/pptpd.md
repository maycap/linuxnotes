##防止用户重复登录pptpd##

* ####前言：
	
	pptpd自身不含有限制账户登录功能，可用pppd功能实现。

* ####以centos6.5为例:


		yum install -y wget perl ppp
	
		rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/pptpd-1.4.0-3.el6.x86_64.rpm
	
	vi /etc/ppp/options.pptpd

		ms-dns 192.168.1.1

	vi /etc/pptpd.conf

		localip 192.168.100.10
		remoteip 192.168.100.60-90

	vi /etc/ppp/chap-secrets

		# client	server	secret			IP addresses
		test		pptpd   test.test	 		* 


* ####NAT转换
		
		1.设置Linux内核支持ip数据包的转发：
			echo "1" > /proc/sys/net/ipv4/ip_forward
		2.加载实现NAT功能必要的内核模块：
			modprobe ip_tables
			modprobe ip_nat_ftp
			modprobe ip_nat_irc
			modprobe ip_conntrack
			modprobe ip_conntrack_ftp
			modprobe ip_conntrack_irc
		3.对iptables中的规则表进行初始化：
			iptables -F
			iptables -X
			iptables -Z
			iptables -F -t nat
			iptables -X -t nat
			iptables -Z -t nat
		4.设置规则链的默认策略：
			iptables -P INPUT ACCEPT
			iptables -P OUTPUT ACCEPT
			iptables -P FORWARD ACCEPT
			iptables -t nat -P PREROUTING ACCEPT
			iptables -t nat -P POSTROUTING ACCEPT
			iptables -t nat -P OUTPUT ACCEPT
		5.添加IP伪装规则：
			iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth1 -j MASQUERADE
			其中：eth1是外网网卡，192.168.1.0/24是内部本地地址，MASQUERADE是符合规则的数据包允许通过
		6.保存
			iptables-save > nat-1.1
			service iptables save

* ####内核转发开启后，直接添加转发规则也可

		iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j SNAT --to-source 外网ip地址/或者内网IP地址

		注: 如果本地需要访问内网，可以设置为内网IP地址，如果是本地需要通过VPN端访问公网，可以设置为外网IP地址


* ####VPN控制

	vpn用户接入时，pppd会调用auth-up和auth-down文件，没有则跳过。centos6.5文件目录为/etc/ppp/，默认没有，可手动添加：


	>1.touch auth-up
	>	
	>2.chmod +x auth-up

	测试pppd传入参数：
	>1.#!/bin/sh
	
	>2.echo $* > arg.list
	
	手动VPN连入测试即可得输入参数为：
	>ppp0 test root /dev/pts/1 115200
	
	引用论坛中的源码解读，在pppd/auth.c的 network_phase()函数里：
	
		1.    /*
	
		2.     * If the peer had to authenticate, run the auth-up script now.
	
		3.     */
	
		4.    if (go->neg_chap || go->neg_upap || go->neg_eap) {
	
		5.        notify(auth_up_notifier, 0);
	
		6.        auth_state = s_up;
	
		7.        if (auth_script_state == s_down && auth_script_pid == 0) {
	
		8.            auth_script_state = s_up;
	
		9.            auth_script(_PATH_AUTHUP);   /* 调用auth-up 脚本 */
	
		10.        }
	
		11.    }
	
	
	在auth.c文件的auth_script()函数里
	
		1.    argv[0] = script;
	
		2.    argv[1] = ifname;
	
		3.    argv[2] = peer_authname;
	
		4.    argv[3] = user_name;
	
		5.    argv[4] = devnam;
	
		6.    argv[5] = strspeed;
	
		7.    argv[6] = NULL;
	
		8. 
	
		9.    auth_script_pid = run_program(script, argv, 0, auth_script_done, NULL);
	
		10. }
	
	
	这里可以看到传递给auth-up/auth-down脚本的参数。
	
	所以，可以利用auth-up脚本来防止用户的重复登录，参考脚本如下：
	
		1. #!/bin/bash
	
		2. # get the username from the parameters
	
		3. USER=$2
	
		4. # if there is a session already for this user, terminate the old one
	
		5. if [ -f /var/run/$USER ]; then
	
		6.    kill -HUP `cat /var/run/$USER`
	
		7.    rm /var/run/$USER
	
		8. fi
	
		9. # remember the pid of the pppd process
	
		10. PPID=`awk '/PPid/ { print $2; }' /proc/$$/status`
	
		11. echo $PPID > /var/run/$USER
	
	
	
	登录测试即可实现限制单独账户功能



