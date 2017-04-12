## lvs-keepalived

### 前言

LVS工作在网络层，通过控制IP来实现负载均衡。IPVS是其具体的实现模块。IPVS的主要作用：安装在Director Server上面，在Director Server虚拟一个对外访问的IP（VIP）。用户访问VIP，到达Director Server，Director Server根据一定的规则选择一个Real Server，处理完成后然后返回给客户端数据。

IPVS有三种机制：NAT，TUN，DR

DR Direct server在VIP:80端口监听用户请求，改写请求报文的MAC地址，将请求负载到real server上，real server将响应直接返回给用户，因此所有的主机必须在同一个网段，且real server可以直接与用户通信。

四种分配方法：

* Round robin (RRS)

将工作平均的分配到服务器 (用于实际服务主机性能一致)

* Least-connections (LCS)

向较少连接的服务器分配较多的工作(IPVS 表存储了所有的活动的连接。用于实际服务主机性能一致。)

* Weighted round robin (WRRS)

向较大容量的服务器分配较多的工作。可以根据负载信息动态的向上或向下调整。 (用于实际服务主机性能不一致时)

* Weighted least-connections (WLC)

考虑它们的容量向较少连接的服务器分配较多的工作。容量通过用户指定的砝码来说明，可以根据装载信息动态的向上或向下调整。(用于实际服务主机性能不一致时)

### 部署
>节点布局

<table>
 <tr>
    <td>名称</td>
	<td>IP</td>
 </tr>
 <tr>
    <td>lvs-master</td>
	<td>192.168.100.80</td>
 </tr>
 <tr>
    <td>lvs-backup</td>
	<td>192.168.100.81</td>
 </tr>
 <tr>
    <td>realserver1</td>
	<td>192.168.100.82</td>
 </tr>
 <tr>
    <td>realserver2</td>
	<td>192.168.100.83</td>
 </tr>
 <tr>
    <td>lvs-vip</td>
	<td>192.168.100.88</td>
 </tr>
</table>

>在realserver绑定虚拟ip地址，脚本如下：

	cat realserver
	#!/bin/bash  
	SNS_VIP=192.168.100.88 
	. /etc/rc.d/init.d/functions  
	case "$1" in  
	start)  
	       ifconfig lo:0 $SNS_VIP netmask 255.255.255.255 broadcast $SNS_VIP  
	       /sbin/route add -host $SNS_VIP dev lo:0  
	       echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore  
	       echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce  
	       echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore  
	       echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce  
	       sysctl -p >/dev/null 2>&1  
	       echo "RealServer Start OK"   
	       ;;  
	stop)  
	       ifconfig lo:0 down  
	       route del $SNS_VIP >/dev/null 2>&1  
	       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore  
	       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce  
	       echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore  
	       echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce  
	       echo "RealServer Stoped"  
	       ;;  
	*)  
	       echo "Usage: $0 {start|stop}"  
	       exit 1  
	esac  
	exit 0 

>keepalived.conf配置文件参考

	! Configuration File for keepalived

	global_defs {
	   notification_email {
	     gencat@163.com
	   }
	   notification_email_from lvs@localhost
	   smtp_server 127.0.0.1
	   smtp_connect_timeout 30
	   router_id LVS_DEVEL
	}
	
	vrrp_instance VI_1 {
	    state MASTER
	    interface eth0
	    virtual_router_id 51
	    priority 100
	    advert_int 1
	    authentication {
	        auth_type PASS
	        auth_pass 1111
	    }
	    virtual_ipaddress {
	        192.168.100.88
	    }
	}
	
	virtual_server 192.168.100.88 80 {  
	    delay_loop 6                    
	    lb_algo wrr                    
	    lb_kind DR                    
	    persistence_timeout 60          
	    protocol TCP                  
	    real_server 192.168.100.82 80 {  
	        weight 3                 
	        TCP_CHECK {  
	        connect_timeout 10         
	        nb_get_retry 3  
	        delay_before_retry 3  
	        connect_port 80  
	        }  
	    }  
	    real_server 192.168.100.83 80 {  
	        weight 3  
	        TCP_CHECK {  
	        connect_timeout 10  
	        nb_get_retry 3  
	        delay_before_retry 3  
	        connect_port 80  
	        }  
	     }  
	} 

>安装lvs-keepalived,采用yum方式，版本已经稳定，采用clush工具

	在 /etc/clustershell/groups添加对应信息，便于统一部署
	lvs: lvs-master lvs-backup 
	lvs-rs: rs[1-2]

	clush -b -w @lvs yum -y install ipvsadm keepalived
	
	#priority 100决定优先级，比如master 100,backup 99
	clush -w lvs-master -c keepalived.conf --dest=/etc/keepalived
	
	#修改priority 99，state MASTER -> BACKUP 复制到 lvs_backup
	clush -w lvs-backup -c keepalived.conf --dest=/etc/keepalived
	
>web服务器简单搭建

	clush -b -w @lvs-rs yum -y install httpd
	clush -b -w @lvs-rs "echo this is test > /var/www/html/index.html"
	clush -b -w @lvs-rs  "hostname >> /var/www/html/index.html"
	clusb -b -w @lvs-rs service httpd start

### 检测

>负载分流检验

	clush -b -w @app curl http://192.168.100.88  

	---------------
	app[13,16,18,201] (4)
	---------------
	This is test
	rs2
	---------------
	app[14-15,17,200] (4)
	---------------
	This is test
	rs1
	
>状态检测

	clush -b -w @lvs ipvsadm -ln

	---------------
	lvs-backup
	---------------
	IP Virtual Server version 1.2.1 (size=4096)
	Prot LocalAddress:Port Scheduler Flags
	  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
	TCP  192.168.100.88:80 wrr persistent 60
	  -> 192.168.100.82:80            Route   3      0          0         
	  -> 192.168.100.83:80            Route   3      0          0         
	---------------
	lvs-master
	---------------
	IP Virtual Server version 1.2.1 (size=4096)
	Prot LocalAddress:Port Scheduler Flags
	  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
	TCP  192.168.100.88:80 wrr persistent 60
	  -> 192.168.100.82:80            Route   3      0          4         
	  -> 192.168.100.83:80            Route   3      0          4   
	
>lvs稳定性检测

	#clush -w lvs-master service keepalived stop
	lvs-master: Stopping keepalived: [  OK  ]

	# clush -b -w @app curl http://192.168.100.88
	---------------
	app[13,15,18,201] (4)
	---------------
	This is test
	rs1
	---------------
	app[14,16-17,200] (4)
	---------------
	This is test
	rs2

	# clush -b -w @lvs ipvsadm -ln

	---------------
	lvs-backup
	---------------
	IP Virtual Server version 1.2.1 (size=4096)
	Prot LocalAddress:Port Scheduler Flags
	  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
	TCP  192.168.100.88:80 wrr persistent 60
	  -> 192.168.100.82:80            Route   3      0          4         
	  -> 192.168.100.83:80            Route   3      0          4         
	---------------
	lvs-master
	---------------
	IP Virtual Server version 1.2.1 (size=4096)
	Prot LocalAddress:Port Scheduler Flags
	  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn

>web节点测试

	# clush -b -w rs1 service httpd stop
	# 对应邮件会收到 => TCP CHECK failed on service <=  报警提示

	# clush -b -w @app curl http://192.168.100.88
	---------------
	app[13-18,200-201] (8)
	---------------
	This is test
	rs2
	
	# clush -b -w @lvs ipvsadm -ln
	---------------
	lvs-backup
	---------------
	IP Virtual Server version 1.2.1 (size=4096)
	Prot LocalAddress:Port Scheduler Flags
	  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
	TCP  192.168.100.88:80 wrr persistent 60
	  -> 192.168.100.83:80            Route   3      0          8         
	---------------
	lvs-master
	---------------
	IP Virtual Server version 1.2.1 (size=4096)
	Prot LocalAddress:Port Scheduler Flags
	  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn