##防止用户重复登录pptpd##

pptpd自身不含有限制账户登录功能，可用pppd功能实现。

以centos6.5为例:
`rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/pptpd-1.4.0-3.el6.x86_64.rpm`

在/etc/ppp/目录或许没有auth-up和auth-down文件：


>1.touch auth-up
>	
>2.chmod +x auth-up

测试pppd传入参数：
>1.#!/bin/sh

>2.echo $* > arg.list

手动VPN连入测试即可得输入参数为：
>ppp0 test root /dev/pts/1 115200

参数二就是登录用户，因此可以利用此参数做限制，参考脚本如下：

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



