## zstack 离线设置



### 故障回溯

>故障版本：

	zstack 1.7
 
>物理节点

	CentOS Linux release 7.2.1511 (Core)

>时间

	Mon Jun 26  CST 2017

>现象

	搬迁机房后，zstack管理页面添加物理机失败。
	
### 解决记录

1、离线环境，推送repo错误

>推送配置修改 stack-redhat.repo

	配置地址：/usr/local/zstack/ansible/files/zstacklib/zstack-redhat.repo
	sed直接替换为内部源即可
	
>创建配置修改 zstack-aliyun-yum.repo
	
	此配置由 /usr/local/zstack/ansible/files/zstacklib/zstacklib.py 直接生成
	sed -i 's/mirrors.aliyun.com/[内部源]/g' 即可

2、依赖安装包错误

>分析

	kvmagent需要qemu-img-ev-2.3.0-31.el7_2.7.1.x86_64 ,结果执行过程过会因为先安装libvirt,而自动安装qemu-img-ev-2.6.0-27.1.el7.x86_64，属于设计逻辑不严谨。
	
>修改推送配置脚本

	位置：/usr/local/zstack/ansible/files/kvm/kvm.py
	修改逻辑: 在安装libvirt前，先指定安装 qemu-img-ev-2.3.0-31.el7_2.7.1.x86_64 即可
	
>大致参考

	115         for pkg in ['qemu-img-ev-2.3.0']:
	116             yum_install_package(pkg, host_post_info)

	