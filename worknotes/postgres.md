##postgres优化##

###前言##
数据库学习笔记

###问题--方法###

####基础配置####
>监听
	
	listen_addresses '*'
	port = 5432 

>连接数

	max_connections = 2000

>访问控制

	vim pg_hba.conf

>共享内存

	shared_buffers = 128MB   

>时区

	timezone = 'Asia/Shanghai'
	log_timezone = 'Asia/Shanghai'
	

#####内存使用率高#####

数据服务器卡，top检测发现，check_point和write占用内存很大，查看资料得知：
checkpoint又名检查点，在oracle中checkpoint的发生意味着之前的脏数据全部写回磁盘，数据库实现了一致性与数据完整性。oracle在实现介质恢复时将以最近的checkpoint为参照点执行事务前滚。在postgresql中checkpoint起着相同的作用：写脏数据；完成数据库的完整性检查。

checkpoints相关参数：

>checkpoint_segments:

WAL log的最大数量，系统默认值是3。该值越大，在执行介质恢复时处理的数据量也越大，时间相对越长。
>checkpoint_timeout:

系统自动执行checkpoint之间的最大时间间隔，同样间隔越大介质恢复的时间越长。系统默认值是5分钟。
>checkpoint_completion_target:

该参数表示checkpoint的完成目标，系统默认值是0.5,也就是说每个checkpoint需要在checkpoints间隔时间的50%内完成。
>checkpoint_warning:

系统默认值是30秒，如果checkpoints的实际发生间隔小于该参数，将会在server log中写入写入一条相关信息。可以通过设置为0禁用信息写入。

>错误锁定：（grep key)

	HINT:  Consider increasing the configuration parameter "checkpoint_segments"
 	LOG:  checkpoints are occurring too frequently (15 seconds apart)

>方法试用

	checkpoint_segments = 5         		# in logfile segments, min 1, 16MB each
	checkpoint_timeout = 6min               # range 30s-1h
	checkpoint_completion_target = 0.9      # checkpoint target duration, 0.0 - 1.0
	checkpoint_warning = 30s                # 0 disables




###TRUOBLE###
>启动不了，去除限制

	#config pgsql env
	echo  'kernel.shmmax = 17179869184' >> /etc/sysctl.conf
	echo  'kernel.shmall = 8388608' >> /etc/sysctl.conf
	echo  'kernel.sem = 500 2048000 128 4096' >> /etc/sysctl.conf
	
	sysctl -p
	
	修改 /etc/security/limits.d/90-nproc.conf  去掉限制
	#*          soft    nproc     1024