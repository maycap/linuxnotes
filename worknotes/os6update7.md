## Centos6—>Centos7

### 整个系统升级步骤（目前由于源缺乏，失败）

> 设置repo

	[upg]
	name=CentOS-$releasever - Upgrade Tool
	baseurl=http://dev.centos.org/centos/6/upg/x86_64/
	gpgcheck=1
	enabled=1
	gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6


> 安装工具

	yum install redhat-upgrade-tool preupgrade-assistant-contents


> 分析系统
	
	preupg -l
	preupg -s CentOS6_7

>添加GPG密钥
	
	rpm --import http://vault.centos.org/centos/RPM-GPG-KEY-CentOS-7
	
>执行更新

	centos-upgrade-tool-cli --force --cleanup-post  --network 7 --instrepo=http://vault.centos.org/centos/7.2.1511/os/x86_64/
>重启

	reboot
	

***

### 只升级内核

>启用ELRepo

	rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
	rpm -Uvh http://www.elrepo.org/elrepo-release-6-6.el6.elrepo.noarch.rpm  
	
>关闭 grub.conf 锁

	chattr -a   /boot/grub/grub.conf

>安装内核（lt=long-term,ml=mainline）

	yum --enablerepo=elrepo-kernel install kernel-lt  


>修改grub.conf,default改成新内核指向的位置

	default=0
	timeout=5
	serial --unit=0 --speed=9600
	terminal --timeout=5 serial console
	title CentOS (3.10.107-1.el6.elrepo.x86_64)
		root (hd0,0)
		kernel /vmlinuz-3.10.107-1.el6.elrepo.x86_64 ro root=UUID=80f436e4-8c35-46c8-a5ad-0f5dd183c68a rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 console=ttyS0 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM
		initrd /initramfs-3.10.107-1.el6.elrepo.x86_64.img
	title CentOS 6 (2.6.32-504.el6.x86_64)
		root (hd0,0)
		kernel /vmlinuz-2.6.32-504.el6.x86_64 ro root=UUID=80f436e4-8c35-46c8-a5ad-0f5dd183c68a rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 console=ttyS0 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM
		initrd /initramfs-2.6.32-504.el6.x86_64.img

>锁定内核

	chattr +a   /boot/grub/grub.conf
	
>重启

	reboot

***

### 故障回顾

>Usage: preupg [options] preupg: error: [Errno 2] No such file or directory: '/root/preupgrade/result.html'

	问题原因：
	openscap 版本过高
	
	解决方法：
	降低 openscap 版本
	yum downgrade openscap
	

>Downloading failed: invalid data in .treeinfo: No option 'upgrade' in section: 'images-x86_64'

	使用http://vault.centos.org/的源