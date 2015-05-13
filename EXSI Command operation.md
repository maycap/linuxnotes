##EXSI命令行摘录##

* 复制镜像：

	>vmkfstools -i XXX.vmdk  Y/Y.vmdk -d thin

* 查看虚拟主机开启:

	>net-stats -l 
	>
	>esxcli  network  vm list

* 虚拟机一些配置操作:

	>esxcli  network ip XX


* 配置网络:
	>esxcfg-vswitch –l (List vSwitch)
	>
	>esxcfg-vswitch –a vSwitch1 (Create vSwitch)
	>
	>esxcfg-vswitch –A “ISCSI” vSwitch1 (Create port group)
	>
	>esxcfg-vswitch –L vmnic1 vSwitch1
	>
	>esxcfg-vmknic -a -i 10.10.10.33 -n 255.255.255.0 ISCSI (Assign IP)
	>
	>esxcfg-vmknic –l (List VMkernelPort) 
 
* 查看网络配置:

	>vim-cmd hostsvc/vmotion/netconfig_get

* 从命令行打开虚拟机电源:

	1. 使用以下命令列出虚拟机的清单 ID：

		>vim-cmd vmsvc/getallvms |grep <vm name>
	
		注意：输出的第一列显示 vmid。


	2. 使用以下命令检查虚拟机的电源状态：

		>vim-cmd vmsvc/power.getstate <vmid>


	3. 使用以下命令打开虚拟机电源

		>vim-cmd vmsvc/power.on <vmid>

