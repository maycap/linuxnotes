## Httpd

### 前言
此等重量级神器，只是简单记录下遇到的使用



### 内存占用控制

过多不用的进程会浪费资源，在httpd.conf配置中有这项，去掉注释即可。
	
	# Server-pool management (MPM specific)
	Include conf/extra/httpd-mpm.conf

查看当前工作模式，httpd -l

	Compiled in modules:
	  core.c
	  mod_so.c
	  http_core.c
	  event.c

>prefork.c  --perfork模式

>worker.c   --worker模式

>event.c    --event模式

打开文件httpd-mpm.conf，对号入座，备注：

MaxConnectionsPerChild   0 #→每个子进程在其生存期内允许伺服的最大请求数量, 如果设置为0,子进程将永远不会结束，为了防止内存泄漏,我们需要将它的默认值不设置为0;
