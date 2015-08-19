##postgres优化##

###前言##
数据库学习笔记

***
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
	
###fsync (boolean)
	
如果这个选项是打开，那么 PostgreSQL 服务器将在好几个地方使用 fsync() 系统调用来确保更新已经物理上写到磁盘中。 这样就保证了数据库集群将在操作系统或者硬件崩溃的情况下恢复到一个一致的状态。

不过，使用 fsync() 会对性能有影响： 在事务提交的时候，PostgreSQL 必须等待操作系统吧预写日志刷新到磁盘上。 在关闭 fsync 的时候，操作系统可以尽可能优化缓冲，排序和推迟写动作。 这样可以显著提高性能。不过，如果系统崩溃，最后提交的几个事务的结果可能部分或者全部丢失。 最糟糕的情况是可能出现不可恢复的崩溃。 

这个选项只能在服务器启动或者 postgresql.conf 文件里设置。 如果这个选项是 off，那么考虑把 guc-full-page-writes 也关闭了。

	

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


***

###TRUOBLE###
>启动不了，去除限制

	#config pgsql env
	echo  'kernel.shmmax = 17179869184' >> /etc/sysctl.conf
	echo  'kernel.shmall = 8388608' >> /etc/sysctl.conf
	echo  'kernel.sem = 500 2048000 128 4096' >> /etc/sysctl.conf
	
	sysctl -p
	
	修改 /etc/security/limits.d/90-nproc.conf  去掉限制
	#*          soft    nproc     1024


***

###实验###

1. pg_dump、pg_restore测试简单性能

>配置

<table>
	<tr>
		<td>item</td>
		<td>配置</td>	
	</tr>
	<tr>
		<td>cpu</td>
		<td> Intel(R) Core(TM) i3-3240 CPU @ 3.40GHz</td>
	</tr>
	<tr>
		<td>cpu MHz</td>
		<td>1600.000</td>
	</tr>
	<tr>
		<td>cache size</td>
		<td>3072 KB</td>
	</tr>
	<tr>
		<td>核数</td>
		<td>4</td>
	</tr>
	<tr>
		<td>memory</td>
		<td>32G</td>
	</tr>
	<tr>
		<td>device rotation rate</td>
		<td>7200 rpm</td>
	<tr>
		<td>postgresql version</td>
		<td>9.3.2</td>
	</tr>
</table>

>文件大小 
	
	5.5G	/web/arh_greenting.tar
>方法

	$ time pg_restore  -p 5433 -d ccpm2 /web/arh_greenting.tar

>结果

<table>
	<tr>
    	<td>\</td>
		<td>参数</td>
		<td>耗时</td>
 	</tr>
 	<tr>
    	<td>item1</td>
		<td>32MB 3 5min 0.9</td>
		<td>real	6m14.161s
			user	0m1.755s
			sys		0m2.819s</td>
 	</tr>
 	<tr>
    	<td>item2</td>
		<td>128MB 3 5min 0.9</td>
		<td>real	5m52.441s
			user	0m1.743s
			sys		0m3.110s
		</td>
 	</tr>
 	<tr>
		<td>item3</td>
		<td>256MB 3 5min 0.9</td>
		<td>real	5m44.444s
			user	0m1.840s
			sys		0m2.969s
		</td>
	</tr>
	<tr>
		<td>item4</td>
		<td>256MB 5 5min 0.9</td>
		<td>real	5m38.562s
			user	0m1.640s
			sys		0m3.179s
		</td>
	</tr>
</table>

日志出现：

	LOG:  checkpoints are occurring too frequently (3 seconds apart)
	HINT:  Consider increasing the configuration parameter "checkpoint_segments".