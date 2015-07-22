##KVM

###前言###
kvm系统采用全虚拟化，在cpu,io采用硬件辅助半虚拟化，性能逼近半虚拟化。


###安装

1.安装kvm

>至少需要安装 qemu-kvm qemu-img 这两个包

	yum install qemu-kvm qemu-img

>安装其他工具包

	yum install virt-manager libvirt libvirt-python python-virtinst libvirt-client

2.安装qemu

>安装编码依赖

	yum install gcc autoconf automake libtool glib* zlib*
	
>从 http://wiki.qemu.org/Download 下载代码

	tar -jzvf qemu-2.3.0.tar.bz2
	cd qemu-2.3.0
	./configure
	make -j 4
	make install

>为方便起见，创建一个link

	ln -s /usr/bin/qemu-system-x86_64 /usr/bin/qemu-kvm

3.配置桥接网络br0

>基础依赖bridge-utils

	yum install bridge-utils

>Step 1:

	cat /etc/sysconfig/network-scripts/ifcfg-eth0
	DEVICE="eth0"
	HWADDR="78:2B:CB:3C:A4:BA"
	NM_CONTROLLD="yes"
	ONBOOT="yes"
	IPADDR=192.168.48.111
	NETMASK=255.255.255.0
	GATEWAY=192.168.48.1

	注释掉 BOOTPROTO
	加入一行
	vim /etc/sysconfig/network-scripts/ifcfg-eth0
	BRIDGE="br0"

>Step 2:

	vim /etc/sysconfig/network-scripts/ifcfg-br0
	DEVICE=br0
	TYPE=Bridge
	ONBOOT=ye
	BOOTPROTO=static
	PREFIX=24
	IPADDR=192.168.48.111
	NETMASK=255.255.255.0
	GATEWAY=192.168.48.1
	STP=on
	DELAY=0

>重启服务

	service network restart

>检测

	#brctl show
		bridge name	bridge id		STP enabled	interfaces
		br0		8000.f8b156d5b77d	yes			em1

4.安装镜像

>virt-install命令方式

	virt-install --name=guest4 --file=/web/images/guest4.dsk --file-size=8 \
	--nonsparse --vcpus=2 --ram=2048 --accelerate \
	--cdrom=/web/CentOS-6.5-x86_64-bin-DVD1.iso  \
	--vnc --vncport=5911 --vnclisten=0.0.0.0  \
	--network bridge=br0 --os-type=linux 
	
	#选择cdrom，在vnc连接装机时，会自动进入cdrom装机界面
	#vncport端口不可重复，每个端口对应为一个kvm镜像

	#--location=http://example1.com/installation_tree/RHEL5.6-Serverx86_64/os 
	#--location=/mnt/centos
	#会选择pxe--装机，选择cobber，可选择性，比ks好控制些
	
>qcow2格式，使用ks安装，启动console

	#创建储蓄镜像，镜像大小是动态增加的，类似于exsi--thin模式。    
    qemu-img create -f qcow2 -o compat=0.10 /web/images/centos63-webtest.img 40G

	#cat create.sh
	virt-install --name=test2  --os-variant=rhel6 \
	--ram 2048 --vcpus=2 --virt-type kvm \
	--disk path=/web/images/test2.img,format=qcow2,size=8,bus=virtio \
	--accelerate --location=http://192.168.100.254 \
	--vnc --vncport=5912 --vnclisten=0.0.0.0 \
	--network bridge=br0,model=virtio --noautoconsole \
	--extra-args='console=tty0 console=ttyS0,115200n8 ks=http://192.168.100.254/ks.cfg'

	#在额外选项中，可设置虚拟机支持console


>报错记录

	'drive-virtio-disk0' uses a qcow2 feature which is not supported by this qemu version: QCOW version 3

	#解决方法，创建时默认格式为1.1，查看如下：
	#qemu-img info /web/images/test1.img 
	image: /web/images/test1.img
	file format: qcow2
	virtual size: 7.0G (7516192768 bytes)
	disk size: 196K
	cluster_size: 65536
	Format specific information:
	    compat: 1.1
	    lazy refcounts: false
	    refcount bits: 16
	    corrupt: false
	
	#修改支持老版，在此创建即可
	qemu-img amend -f qcow2 -o compat=0.10 test.qcow2

	

5.vnc连接装机
	
	#下载vnc -- http://vnc.en.softonic.com/
	输入IP：vncport，即可连接对应kvm镜像



6.修改支持 virsh console
	
>添加ttyS0:

	echo "ttyS0" >> /etc/securetty

>在/etc/grub.conf文件中为内核添加参数:	

	console=ttyS0

>在/etc/inittab中添加agetty:

	S0:12345:respawn:/sbin/agetty ttyS0 115200

>重启客户机，连接

	virsh console guest4
	


	
	