## supervisor

###简介###

Supervisor是一个进程管理工具：

用途就是有一个进程需要每时每刻不断的跑，但是这个进程又有可能由于各种原因（此次背景就是一个辅助应用内存会一直增大，加入了cgroup控制了，达到设置线会被kill，需要重启）有可能中断。当进程中断的时候我希望能自动重新启动它，此时，我就需要使用到了Supervisor。


###安装

	yum  install supervisor

###配置

>配置文件 /etc/supervisord.conf


举一例说明

	vim /etc/supervisord.conf

	[program:Jtoy]
	command=python /home/web/jtoy/Jtoy.py &
	autostart=true
	autorestart=true
	user=web
	log_stderr=true
	logfile=/home/web/jtoy/Jtoy.log

	
	#备注
	由于Jtoy.py是前台启动，所以需要放入&，进入后台，才能自启动成功

>重启supervisor

	/etc/init.d/supervisord restart

	

	