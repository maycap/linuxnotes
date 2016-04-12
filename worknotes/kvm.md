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

>查看执行系统格式

	virt-install --os-variant list


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
	
7.导出XML配置，修改，生成

	#导出域配置文件
	virsh dumpxml  test4  > /web/test4.xml 

	<domain type='kvm' id='14'>       --id,删除自动生成
	  <name>test6</name>              --名字不可重
	  <memory unit='KiB'>4194304</memory>    --唯一，删掉，自动创建
	  <currentMemory unit='KiB'>4194304</currentMemory>
	  <vcpu placement='static'>2</vcpu>
	  <os>
	    <type arch='x86_64' machine='rhel6.6.0'>hvm</type>
	    <boot dev='hd'/>
	  </os>
	  <features>
	    <acpi/>
	    <apic/>
	    <pae/>
	  </features>
	  <clock offset='localtime'/>
	  <on_poweroff>destroy</on_poweroff>
	  <on_reboot>restart</on_reboot>
	  <on_crash>restart</on_crash>
	  <devices>
	    <emulator>/usr/libexec/qemu-kvm</emulator>
	    <disk type='file' device='disk'>
	      <driver name='qemu' type='qcow2' cache='none'/>
	      <source file='/web/images/test6.img'/>        --指定虚拟磁盘位置
	      <target dev='vda' bus='virtio'/>
	      <alias name='virtio-disk0'/>
	      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
	    </disk>
	    <controller type='usb' index='0' model='ich9-ehci1'>
	      <alias name='usb0'/>
	      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x7'/>
	    </controller>
	    <controller type='usb' index='0' model='ich9-uhci1'>
	      <alias name='usb0'/>
	      <master startport='0'/>
	      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0' multifunction='on'/>
	    </controller>
	    <controller type='usb' index='0' model='ich9-uhci2'>
	      <alias name='usb0'/>
	      <master startport='2'/>
	      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x1'/>
	    </controller>
	    <controller type='usb' index='0' model='ich9-uhci3'>
	      <alias name='usb0'/>
	      <master startport='4'/>
	      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x2'/>
	    </controller>
	    <interface type='bridge'>
	      <mac address='52:54:00:11:22:33'/>     --mac重复会导致ip一样而不会报错，前六位不动，修改后六位
	      <source bridge='br0'/>
	      <target dev='vnet4'/>                  --网络配置，一下都可删掉，防止冲突
	      <model type='virtio'/>				 	
	      <alias name='net0'/>
	      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
	    </interface>
	    <serial type='pty'>
	      <target port='0'/>
	      <alias name='serial0'/>
	    </serial>
	    <console type='pty' tty='/dev/pts/6'>   --具体指定删掉
	      <target type='serial' port='0'/>
	      <alias name='serial0'/>
	    </console>
	    <input type='tablet' bus='usb'>
	      <alias name='input0'/>
	    </input>
	    <input type='mouse' bus='ps2'/>
	    <graphics type='vnc' port='5915' autoport='no' listen='0.0.0.0'>
	      <listen type='address' address='0.0.0.0'/>    --vnc端口不可重
	    </graphics>
	    <video>                                         --不用视频即可删掉多余
	      <model type='cirrus' vram='9216' heads='1'/>
	      <alias name='video0'/>
	      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
	    </video>
	    <memballoon model='virtio'>
	      <alias name='balloon0'/>
	      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
	    </memballoon>
	  </devices>
	</domain>

	
8.动态添加磁盘

	#添加iso，指定挂载为vdd
	virsh # attach-disk xp2 /code/kvm/NetKVM.iso vdd

9.修改配置文件添加

	<disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <source file='/code/kvm/NetKVM.iso'/>
      <target dev='hdc' bus='ide'/>
      <readonly/>
      <alias name='ide0-1-0'/>
      <address type='drive' controller='0' bus='1' target='0' unit='0'/>
    </disk>

	#动态添加未必识别，指定配置文件需要先关闭，start or create  xp2 才可获取


***

>安装xp网卡不识别


	#下载网卡驱动，挂载上去，更新网卡驱动即可
	http://www.famzah.net/download/kvm/virtio-windows/24.09.2009/NetKVM.iso

>exsi5.5 二次虚拟化

	1. egrep -c '(vmx|svm)' /proc/cpuinfo 
	查看cpu是否支持虚拟化

	2. 关闭虚拟机

	3. vim  kvm1.vmx
	 
	#尾行计入四行配置，开启cpu虚拟化
	nce.enable = TRUE
	hypervisor.cpuid.v0 = FALSE
	featMask.vm.hv.capable ="Min:1"
	vhv.enable= TRUE

	4. 启动虚拟机
	
	5. lsmod | grep kvm
	检测kvm模块是否加载
	



	

	