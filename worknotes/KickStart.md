## Kickstart无人值守安装

* #### PXE概念
	
	严格来说，PXE 并不是一种安装方式，而是一种引导的方式。进行 PXE 安装的必要条件是要安装的计算机中包含一个 PXE 支持的网卡（NIC），即网卡中必须要有 PXE 客户端。PXE （Pre-boot Execution Environment，直译为预启动执行环境）协议使计算机可以通过网络启动。协议分为 client 和 server 端，PXE client 在网卡的 ROM 中，当计算机引导时，BIOS 把 PXE client 调入内存执行，由PXE client 将放置在远端的文件通过网络下载到本地运行。运行 PXE 协议需要设置 DHCP服务器和 TFTP 服务器。DHCP 服务器用来给PXE client（将要安装系统的主机）分配一个IP 地址，由于是给 PXE client 分配 IP 地址，所以在配置 DHCP 服务器时需要增加相应的PXE 设置。此外，在 PXE client 的 ROM 中，已经存在了 TFTP Client。PXE Client 通过TFTP 协议到 TFTP Server 上下载所需的文件。

* #### 设备
	>DHCP server

	>TFTP server

	>ks.cfg configure file

	>iso server,like NFS HTTP or FTP

	
* #### HTTP SERVER

	
		1.yum install  httpd*
		2.mount /path/*.iso /mnt/iso -t iso9660 -o loop
		3.cp -a /mnt/iso/* /var/html/www

	
* #### TFTP SERVER


		1.yum install tftp-server
		2.vi /etc/xinetd.d/tftp
			service tftp
			{
				socket_type		= dgram
				protocol		= udp
				wait			= yes
				user			= root
				server			= /usr/sbin/in.tftpd
				server_args		= -s /tftpboot
				disable			= no
				per_source		= 11
				cps			= 100 2
				flags			= IPv4
			}
		3.service xinetd restart
		4.mkdir /tftpboot
		5.cp /usr/share/syslinux/pxelinux.0 /tftpboot
	 	6.cp /var/html/www/images/pxeboot/initrd.img /tftpboot
		7.cp /var/html/www/images/pxeboot/vmlinuz /tftpboot
		8.cp /var/html/www/isolinux/*.msg /tftpboot
		9.mkdir /tftpboot/pxelinux.cfg
	 	10.cp /var/html/www/isolinux/isolinux.cfg /tftpboot/pxelinux.cfg/default
		
		cat default
			default linux
			prompt 1
			timeout 6
			display boot.msg
			F1 boot.msg
			F2 options.msg
			F3 general.msg
			F4 param.msg
			F5 rescue.msg
			label linux
			kernel vmlinuz
			append initrd=initrd.img ks=http://192.168.100.254/ks.cfg

	
* #### DHCP SERVER

		yum install dhcp
		cat /etc/dhcp/dhcpd.conf
		  	allow booting;
		    allow bootp;
		    # A slightly different configuration for an internal subnet.
		    subnet 192.168.100.0 netmask 255.255.100.0 {
		    range 192.168.100.110 192.168.100.120;
		    # option domain-name-servers yourdomain;
		    option domain-name "yourdomain";
		    option routers 192.168.1.1;
		    default-lease-time 600;
		    max-lease-time 7200;
		    filename "/pxelinux.0";
		    next-server 192.168.100.254;
		    }
	
	在/etc/init.d/dhcpd 默认启动用户是dhcpd，若是启动不了，改成root即可
	
		cat /etc/init.d/dhcpd
		user=root
		group=root

* #### ks.cfg
	
		cat /var/www/html/ks.cfg
			#platform=x86, AMD64, or Intel EM64T
			#version=DEVEL
			key --skip
			# Firewall configuration
			firewall --disabled
			# Install OS instead of upgrade
			install
			# Use network installation
			#nfs --server=192.168.100.254 --dir=/media
			url --url=http://192.168.100.254/
			# Root password
			rootpw  --iscrypted $6$gJz9rJzgVM0h7TvX$LHJz06m/P6CfEjQifcrSOKy6UDc55RvfRzebSmn7RCeUmJvZ4FLR.oZWsdXqkC3VPgJ9OyW2rzoEbMdyT8t/X.
			# Network information,not point interface is good chioce
			#network  --bootproto=dhcp --device=eth0 --onboot=on
			network  --bootproto=dhcp --onboot=on
			# System authorization information
			auth  --useshadow  --passalgo=md5
			# Use text mode install
			text
			# System keyboard
			keyboard us
			# System language
			# lang zh_CN
			lang en_US.UTF-8
			# SELinux configuration
			selinux --disabled
			# Do not configure the X Window System
			skipx
			# Installation logging level
			logging --level=info
			# Reboot after installation
			reboot
			# System timezone
			timezone  Asia/Shanghai
			# System bootloader configuration
			bootloader --location=mbr
			# Clear the Master Boot Record
			zerombr
			# Partition clearing information
			clearpart --all  
			# Disk partitioning information
			part swap --size 4096 
			part /boot --size 200 
			part / --fstype=ext4 --size 30960
			part pv.01 --grow --size=200
			volgroup vg_21tb  pv.01
			#logvol /web --vgname=vg_21tb --size=819200 --name=DATA
			logvol /web --vgname=vg_21tb --size=500 --name=DATA
			 
			%post
			#wget ftp://192.168.100.254/pub/yum.repo -P /etc/yum.repos.d/
			%packages
			@core
			@server-policy
			@workstation-policy
			%end
	
* #### PXE 引导启动即可

***

### 感慨：
这种使用率不高的系统，再次使用时，请确保 xinetd，dhcpd,httpd服务是开启的。然后装机仍是出现问题，请相信过去的成功是真实存在的，然后确认iptables,selinux这两货的存在！

	service iptables stop
	setenforce 0
	
		
	