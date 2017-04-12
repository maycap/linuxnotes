## cgroup

### 简介

Cgroups是control groups的缩写，是Linux内核提供的一种可以限制、记录、隔离进程组（process groups）所使用的物理资源（如：cpu,memory,IO等等）的机制。

实验环境参考：Centos6.5

>概念


- Subsystems: 称之为子系统，一个子系统就是一个资源控制器，比如 cpu子系统就是控制cpu时
间分配的一个控制器。


- Hierarchies: 可以称之为层次体系也可以称之为继承体系，指的是Control Groups是按照层次体系的关系进行组织的。

- Control Groups: 一组按照某种标准划分的进程。进程可以从一个Control Groups迁移到另外一个
Control Groups中，同时Control Groups中的进程也会受到这个组的资源限制。

- Tasks: 在cgroups中，Tasks就是系统的一个进程。


>子系统

- blkio这个子系统为块设备设定输入/输出限制，比如物理设备（磁盘，固态硬盘，USB 等等） 。
- cpu这个子系统使用调度程序提供对 CPU 的 cgroup 任务访问。
- cpuacct这个子系统自动生成 cgroup 中任务所使用的 CPU 报告。
- cpuset这个子系统为 cgroup 中的任务分配独立 CPU（在多核系统）和内存节点。
- devices这个子系统可允许或者拒绝 cgroup 中的任务访问设备。
- freezer这个子系统挂起或者恢复 cgroup 中的任务。
- memory这个子系统设定 cgroup 中任务使用的内存限制，并自动生成由那些任务使用的内存资源报告。
- net_cls这个子系统使用等级识别符（classid）标记网络数据包，可允许 Linux 流量控制程序（tc）识别从具体 cgroup 中生成的数据包。
- ns名称空间子系统。

>官方规则

1. 一个单继承体系(单层次体系)可以附加1个或者多个子系统
2. 一个子系统，不能被附加到多个继承体系中。
3. 每当在系统中创建一个继承体系的时候，会默认再创建一个control groups，并且这个
control groups被称之为root cgroup，此时整个系统中的tasks(进程)都属于这个root cgroup。
系统中的进程，在一个继承体系中都明确的属于一个control groups，并且这个进程可以从
一个control groups移动到另外一个control groups中，但是需要主要的是，在一个继承体系中
一个进程是没办法同时属于两个control groups的，但是一个进程可以同时属于两个不同的继承
体系中的control groups。
4. 系统上的任何task(进程)通过fork创建子task(进程)的时候，这个子task(进程)，自动继承
其父task(进程)的control groups，成为这个control groups的一员。
此后这个子task(进程)可以移动到其他control groups中，父task(进程)和子task(进程)完全独立。

>自我总结规则两条

1. rule1-3只为说一件事：一个进程只需要被同一类别的子系统限制一次，不同类别互相独立。其中所引入的体系是为了整合配置（默认一个子系统对应一个体系），cgroup为了按需配置！
2. 子进程默认继承，不想就换。


###实战

>安装

	yum  install libcgroup

>挂载抽象子系统为（伪文件）体系

	cgconfig默认配置为 /etc/cgconfig.conf

	mount {
		cpuset	= /cgroup/cpuset;
		cpu	= /cgroup/cpu;
		cpuacct	= /cgroup/cpuacct;
		memory	= /cgroup/memory;
		devices	= /cgroup/devices;
		freezer	= /cgroup/freezer;
		net_cls	= /cgroup/net_cls;
		blkio	= /cgroup/blkio;
	}

	#每个子系统为一个体系
	#启动程序，挂载生效
	/etc/init.d/cgconfig  start    

	#查看挂载体系
	ls  /cgroup 

>整合子系统，自行挂载

1. 先关闭系统自动挂载

	/etc/init.d/cgconfig stop

2. 建立体系挂载目录，一般都放在 /cgroup下

	mkdir  /cgroup/cap

3. 挂载子系统到体系（伪文件目录）

	--体系名和目录名一致便于记忆

	mount -t cgroup -o cpu,cpuset cap /cgroup/cap/

4. 增删体系中的子系统

	mount -t cgroup -o remount,cpu cap /cgroup/cap/

5. 卸载挂载体系

	umount /cgroup/cap/

6. 查看已挂载体系

	lssubsys 

	or

	cat /proc/mounts


>cgroup

1. 创建语法

	cgcreate -t uid:gid -a uid:gid -g subsystems: path

	uid是指用户名

	-t 可选，来指定用户或者组对这个cgroup的tasks伪文件的拥有权，仅允许指定的用户或者组，往这个cgroup中添加tasks。

	-a 可选，用来指定用户和组对这个cgroup的除了tasks外的其他伪文件的拥有权。用来修改cgroup中指定的tasks可以访问的系统资源。

	linux的伪文件系统，其实权限就是对应文件属主。


	cgcreate -t nagios:nagios -a nagios:nagios -g cpu,cpuset:/nagiosgroup
	
2. 移除

	cgdelete  -g cpu:/nagiosgroup

3. 调节参数

	cgset -r memory.limit_in_bytes=10240000 mygroup

	cgset --copy-from group1/ group2/

	cgget -r memory.limit_in_bytes mygroup

	or

	cat /cgroup/mem/mygroup/memory.limit_in_bytes 

4. 查看cgroup

	lscgroup

5. 查看进程使用cgroup

	ps -O cgroup  -e


>对进程控制task

1. 手动加入pid

	a)echo 1701 > /cgroup/mem/mygroup/tasks

	b)cgclassify -g cpu,memory:mygroup 1701 1138

2. 自动时加入

	cgexec -g memory:/pythongroup  python /home/web/jtoy/Jtoy.py &
	
	
	sh -c "echo \$$ > /cgroup/cpu_and_mem/group1/tasks &&  python /home/web/jtoy/Jtoy.py &"


>配置快照

	手动添加如何自动保存为配置，可以使用 cgsnapshot

	cgsnapshot > /tmp/xxxxx.conf


### 写入配置

手动调节很有成就感，不过一旦重启就要再来一遍，很容易忘记。因此写入配置文件，一目了然。

	cat /etc/cgconfig.conf 

	mount {
		cpuset	= /cgroup/cpuset;
		cpu	= /cgroup/cpu;
		cpuacct	= /cgroup/cpuacct;
		memory	= /cgroup/memory;
		devices	= /cgroup/devices;
		freezer	= /cgroup/freezer;
		net_cls	= /cgroup/net_cls;
		blkio	= /cgroup/blkio;
	}
	

	#指定task属主，admin也必须指定
	group mygroup{
		perm  {
			task {
				uid = web;
				gid = web;
			}admin{
				uid = root;
				gid = root;
			}
		}
	
		memory{
			memory.limit_in_bytes = "102400000";
		}
	}

	
	group mygroup/python {
		memory {
			 memory.limit_in_bytes = "20480000";
		}
	}



手动加入控制进程太过于繁琐，采用cgred程序

	cat /etc/cgrules.conf

	web		memory	  	/mygroup/python	

	web:ftp  devices  /mygroup/ftp

	#举例说明
	@web  devices  /mygroup
	@dev  %  %
	
	-------通配符说明-----
	@ -- 是用户组
	* -- 匹配所有
	% -- 表示条目和上面的一样

	




	





	 







