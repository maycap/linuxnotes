##防止用户重复登录pptpd##

pptpd自身不含有限制账户登录功能，可用pppd功能实现。

以centos6.5为例:

`rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/pptpd-1.4.0-3.el6.x86_64.rpm`

vpn用户接入时，pppd会调用auth-up和auth-down文件，没有则跳过。centos6.5文件目录为/etc/pp/，默认没有，可手动添加：


>1.touch auth-up
>	
>2.chmod +x auth-up

测试pppd传入参数：
>1.#!/bin/sh

>2.echo $* > arg.list

手动VPN连入测试即可得输入参数为：
>ppp0 test root /dev/pts/1 115200

引用论坛中大神给出的源码解读，在pppd/auth.c的 network_phase()函数里：

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



