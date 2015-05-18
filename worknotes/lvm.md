###LVM notes###

***
###前言###

LVM系统教程很多，只记下常使用的一些组合命令以及遇到的坑。
***

####减少分区大小，增加新分区
	1.umount /dev/vg_eln4/web
	  
	2.e2fsck -f /dev/mapper/vg_eln4-web
	  resize2fs /dev/vg_eln4/web 100G
	#从原分区中去掉磁盘空间用用
	
	3.lvreduce -L -10G /dev/vg_eln4/web
	
	4.lvcreate -L 10G -n mfs_data vg_eln4
	
	5.mkfs.ext4 /dev/vg_eln4/mfs_data
	
	6.mkdir /usr/local/mfs
	
	7.fstab
		/dev/vg_eln4/mfs_data /usr/local/mfs    ext4  defaults   0 0
		#开机lvm检测失败导致启动异常，使用 0 0，跳过检测
	
	8.mount -a

####在线添加分区
	e2fsck -f /dev/vg_eln4/web
	lvextend -l +1000 /dev/vg_eln4/web
	resize2fs /dev/vg_eln4/web

####卸载添加分区
	umount /web
	e2fsck -f /dev/vg_eln4/web
	lvextend -l +1000 /dev/vg_eln4/web
	resize2fs /dev/vg_eln4/web
	mount -a
	#速度较快

***
###修改lvm导致系统起不来
	由于lvm误操作导致文件系统检测不过
	/dev/mapper/vg_eln4-web /web   ext4    defaults        1 2
	就是2指示的文件系统检测错误导致

	插入光盘，选择修复模式
	选择命令行模式
	chroot  /dev/sysimage
	vim /etc/fstab
	ctrl-D
	reboot




