###Linux Net Speed###

***
###前言###
现在平台服务器前端文件占用带宽很大，导致加载缓慢。再加上视频服务，已经捉襟见肘，参考实验做此笔记，以备不时之需


###修改内核加速

>net.ipv4.tcp_window_scaling

	启用 Window Scaling可以支持最大1GB的窗口，否则只支持 64KB 的窗口。
	
	查看当前是否启用：
		sysctl net.ipv4.tcp_window_scaling
	使用这个命令启用：
		sysctl -w net.ipv4.tcp_window_scaling=1

>initcwnd

	在Linux 2.6.33~2.6.38系统上initcwnd需要手工增加到10。Linux 2.6.39, RHEL 6.2开始默认就是10，提供网络吞吐
	
	查看方式：
		ip route show
		ss -nlit

	脚本修改：
		ip route | while read p; do 
		> ip route change $p initcwnd 10
		> done

>net.ipv4.tcp_congestion_control

	拥塞预防算法把丢包或网络延迟增大作为网络拥塞的标志，即路径中某个连接或路由器已经拥堵了，以至于必须采取删包措施。
	因此，必须调整窗口大小，以避免造成更多的包丢失，从而保证网络畅通。

	常用的拥塞控制算法：
	cubic：当前Linux内核默认算法，能在多条共享瓶颈链路的TCP连接之间保持良好的RTT公平性。
	htcp：高性能网络中综合表现比较优秀的算法，需要手工加载。
	hybla：在卫星链路等高延迟、高丢包率环境中优秀的算法，需要手工加载。
		
	加载模块：
	modprobe tcp_htcp

	查看当前可用拥塞控制算法：
	sysctl net.ipv4.tcp_available_congestion_control
	修改拥塞控制算法：
	sysctl -w net.ipv4.tcp_congestion_control=htcp