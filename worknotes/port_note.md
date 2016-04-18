##端口复用和转发


###端口复用笔记

>haproxy

	frontend testport
	    mode tcp
	    bind *:80
	
	    tcp-request inspect-delay 3s
	    acl is_http req_proto_http
	    acl is_https req_ssl_ver 2:3.1
	    tcp-request content accept if is_http
	    use_backend http if is_http
	    use_backend https if is_https
	    use_backend ssh if !is_http
	
	
	backend ssh
	    mode tcp
	    server ssh01 127.0.0.1:22 maxconn 10 check inter 3s
	
	backend http
	    mode tcp
	    server ngx01 127.0.0.1:8080 maxconn 10 check inter 3s
	
	backend https
	    mode tcp
	    server ngx01 127.0.0.1:443 maxconn 10 check inter 3s

>sslh

	https://github.com/yrutschle/sslh


###端口转发

>iptables(linux)

	1. 需要先开启linux的数据转发功能
	vim /etc/sysctl.conf
	将net.ipv4.ip_forward=0改为1
	sysctl -p

	2.开启远程桌面

	A为外网机器，外网IP：NETIPA，内网IP：INNERIPA
	B为内网机器，内网IP：INNERIPB

	# cat win10.sh 
	iptables -t nat -A PREROUTING -d NETIPA -p tcp --dport 3389 -j DNAT --to-destination INNERIPB:3389
	------转发入口应写为外网IP地址
	
	iptables -t nat -A POSTROUTING -d INNERIPB -p tcp --dport 3389 -j SNAT --to-source INNERIPA
	-----转发出口的入口由于同在内网，需写成内网IP地址

	3.删除转发
	#查看序号
	iptables -t nat -nL --line-number	
	
	#删除对应序号
	iptables -t nat -D PREROUTING 2
	


>netsh(windows)

	1.首先安装IPV6（xp下IPV6必须安装，否则端口转发不可用！）
	netsh interface ipv6 install

	2.添加一个IPV4到IPV4的端口映射
	netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=xxx.xxx.xxx.xxx connectport=22

	3.指定监听ip和端口可以删除
	netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=22

	4.可以查看存在的转发
	netsh interface portproxy show all

>ssh（通用）

	A机器：ipA
	B机器：ipB
	C机器：ipC

	A只能访问B，B能访问C

	-L  本地转发（ A:4000 = c:80)
	A> ssh -p 22 -CfNg -L 0.0.0.0:4000:ipC:80  user@IpB
	#用A的4000端口，通过B的22端口，转发给C的80端口(前提就是B可以访问C的80）
	
	-R 远程转发（ C:8000 = A:1234)
	A> ssh -p 22 -CfNg -R IPC:8000:ipA:1234 user@IPB
	#C开启8000，代理A的1234
	
	#名词解释：
	T表示不使用伪终端
	C表示压缩数据传输
	f表示后台用户验证,这个选项很有用,没有shell的不可登陆账号也能使用.
	N表示不执行脚本或命令
	g表示允许远程主机连接转发端口

>端口端口转发（通道）

	-D 建立动态SOCKS4/5的代理通道
	ssh -CfNg -D 1234 user@ip  

>autossh（自动重连）

	#调用ssh,封装一层而已，用法参考如下（ -M 启用的监听ssh端口，会占用5678和5679）
	autossh -M 5678 -NR 41123:localhost:22 web@IP  -f -p 8000