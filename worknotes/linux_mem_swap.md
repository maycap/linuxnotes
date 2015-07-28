##linux_mem_swap##

###前言###
与linux_skill并不同，此笔记主要记录linux上关于内存和交换分区。

***

###buffer和cache

cache可以说是个混合大名词了，旗下有三大分支：cpu\_cache、page\_cache、buffer\_cache


>cpu_cache:高速缓存，是位于CPU与主内存间的一种容量较小但速度很高的存储器

由于CPU的速度远高于主内存，CPU直接从内存中存取数据要等待一定时间周期，Cache中保存着CPU刚用过或循环使用的一部分数据，当CPU再次使用该部分数据时可从Cache中直接调用,这样就减少了CPU的等待时间,提高了系统的效率。

Cache又分为一级Cache(L1 Cache)和二级Cache(L2 Cache)，L1 Cache集成在CPU内部，L2 Cache早期一般是焊在主板上,现在也都集成在CPU内部，常见的容量有256KB或512KB L2 Cache。	

>page_cache:vfs文件系统层的cache


对于一个ext3文件系统而言，每个文件都会有一棵radix树管理文件的缓存页，这些被管理的缓存页被称之为page cache。所以，page cache是针对文件系统而言的。例如，ext3文件系统的页缓存就是page cache。

>buffer_cache:是针对设备的，每个设备都会有一棵radix树管理数据缓存块

为了提高磁盘设备的IO性能，我们采用内存作为磁盘设备的cache。用户操作磁盘设备的时候，首先将数据写入内存，然后再将内存中的脏数据定时刷新到磁盘。这个用作磁盘数据缓存的内存就是所谓的buffer cache。

>buffer\_cache  VS   page\_cache

在以前的Linux系统中，有很完善的buffer cache软件层，专门负责磁盘数据的缓存。在磁盘设备的上层往往会架构文件系统，为了提高文件系统的性能，VFS层同样会提供文件系统级别的page cache。这样就导致系统中存在两个cache，并且重叠在一起，显得没有必要和冗余。为了解决这个问题，在现有的Linux系统中对buffer cache软件层进行了弱化，并且和page cache进行了整合。Buffer cache和page cache都采用radix tree进行维护，只有当访问裸设备的时候才会使用buffer cache，正常走文件系统的IO不会使用buffer cache。

>free中的buffer和cache

	buffer : 作为buffer cache的内存，是块设备的读写缓冲区，更靠近存储设备，或者直接就是disk的缓冲区。
    cache: 作为page cache的内存, 文件系统的cache，是memory的缓冲区。

 如果 cache 的值很大，说明cache住的文件数很多。如果频繁访问到的文件都能被cache住，那么磁盘的读IO 必会非常小。

###free###
如此常见的命令，直接上数据如下：

	# free -m
            	 total       used       free     shared    buffers     cached
	Mem:         32107      31726        380          0        821       6777
	-/+ buffers/cache:      24127       7979
	Swap:            0          0          0

附选项m，代表单位是MB。在linux的内存分配机制中，优先使用物理内存，当物理内存还有空闲时（还够用），不会释放其占用内存，就算占用内存的程序已经被关闭了，该程序所占用的内存用来做缓存使用，对于开启过的程序、或是读取刚存取过得数据会比较快。各项说明如下：

>整体说明

	Mem：表示物理内存统计。
	-/+ buffers/cached：表示物理内存的缓存统计 
	Swap：表示硬盘上交换分区的使用情况。只有mem被当前进程实际占用完,即没有了buffers和cache时，才会使用到swap。

>Mem 行（第一行）数据说明：

	Total：32107MB。表示物理内存总大小。
	Used：31726MB。表示总计分配给缓存（包含buffers 与cache ）使用的数量，但其中可能部分缓存并未实际使用。
	Free：380MB。表示未被分配的内存。
	Shared：0MB。共享内存，一般系统不会用到。
	Buffers：821MB。系统分配但未被使用的buffers 数量。
	Cached：6777MB。系统分配但未被使用的cache 数量。
 
>-/+ buffers/cache 行（第二行）数据说明：

	Used：24127MB，实际使用的buffers 与cache 总量，也是实际使用的内存总量。
	Free: 7979MB, 未被使用的buffers 与cache 和未被分配的内存之和，这就是系统当前实际可用内存。

>分析结论等式

1.实际可用内存大小：
	
	Free（-/+ buffers/cache行）= Free(Mem)+buffers(Mem)+Cached(Mem);
	7979 = 380 + 821 + 6777

2.已经分配的内存大小：

	Used(Mem) = Used(-/+ buffers/cache)+ buffers(Mem) + Cached(Mem)
	31726 = 24127 + 821 + 6777

3.物理内存总大小：
	
	total（Mem） = used(-/+ buffers/cache) + free(-/+ buffers/cache)
	32107 = 31726 + 7979


***
###top
直接上数据如下：
	
	  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  DATA COMMAND                                                                           
	11572 web       20   0 5430m 1.0g  10m S 133.7  3.1   3492:53 5.1g java                                                                              
	12753 web       20   0 5458m 1.0g 7020 S 122.3  3.2   6232:28 5.2g java    

>抽取内存介绍

>VIRT：virtual memory usage 虚拟内存
	
	1、进程“需要的”虚拟内存大小，包括进程使用的库、代码、数据等
	2、假如进程申请100m的内存，但实际只使用了10m，那么它会增长100m，而不是实际的使用量
	3、VIRT = ps -o size --no-header $pid

>RES：resident memory usage 常驻内存

	1、进程当前使用的内存大小，但不包括swap out
	2、包含其他进程的共享
	3、如果申请100m的内存，实际使用10m，它只增长10m，与VIRT相反
	4、关于库占用内存的情况，它只统计加载的库文件所占内存大小

>SHR：shared memory 共享内存

	1、除了自身进程的共享内存，也包括其他进程的共享内存
	2、虽然进程只使用了几个共享库的函数，但它包含了整个共享库的大小
	3、计算某个进程所占的物理内存大小公式：RES – SHR
	4、swap out后，它将会降下来

>DATA： 等价于VIRT

	1、数据占用的内存。如果top没有显示，按f键可以显示出来。
	2、真正的该程序要求的数据空间，是真正在运行中要使用的。
		
>%MEM：进程使用的物理内存百分比

	RES = Mtotal * %MEM


###swap

如果系统的物理内存用光了，则会用到swap。系统就会跑得很慢，但仍能运行;如果Swap空间用光了，那么系统就会发生错误。通常会出现“application is out of memory”的错误，严重时会造成服务进程的死锁。

>设置使用,在/etc/sysctl.conf

	#当内存未被征用量不足10%时（而不是实际使用量），开始使用swap分区
	vm.swappiness = 10

>动态添加swap

	#创建用于交换分区的文件
	dd if=/dev/zero of=/web/swap_mount_4G bs=1024 count=4096000

	#设置交换分区文件
	mkswap /swap_mount_4G

	#立即启用交换分区文件
	swapon /swap_mount_4G 

	#若要想使开机时自启用，则需修改文件/etc/fstab中的swap行
	/web/swap_mount_4G swap swap defaults 0 0


>查看命令：
	
	free,top,vmstat

>/proc/1/smaps项目字段解读

	Size:                 36 kB     --是进程使用内存空间，并不一定实际分配了内存(VSS)
	Rss:                   0 kB		--是实际分配的内存(不需要缺页中断就可以使用的)
	Pss:                   0 kB		--是平摊计算后的使用内存(有些内存会和其他进程共享，例如mmap进来的)
	Shared_Clean:          0 kB		--和其他进程共享的未改写页面
	Shared_Dirty:          0 kB		--和其他进程共享的已改写页面
	Private_Clean:         0 kB		--未改写的私有页面页面
	Private_Dirty:         0 kB		--已改写的私有页面页面
	Referenced:            0 kB		--当前被标记为引用或访问的内存量
	Anonymous:             0 kB		--显示不属于任何文件的内存量
	AnonHugePages:         0 kB		--支持非文件的大内存页映射到用户空间的页表
	Swap:                  0 kB		--存在于交换分区的数据大小(如果物理内存有限，可能存在一部分在主存一部分在交换分区)
	KernelPageSize:        4 kB		--操作系统一个页面大小
	MMUPageSize:           4 kB		--体系结构MMU一个页面大小
	Locked:                0 kB
	VmFlags: rd ex mr mw me dw 

	#插曲 Rss = Shared_Clean + Shared_Dirty + Private_Clean + Private_Dirty

“VmFlags”字段需要单独说明。该成员表示内核与所述特定虚拟存储器区域中两字母编码相关联的标志方式。标识码如下：（不同版本，标志意思可能不同）

	rd  - readable
	wr  - writeable
	ex  - executable
	sh  - shared
	mr  - may read
	mw  - may write
	me  - may execute
	ms  - may share
	gd  - stack segment growns down
	pf  - pure PFN range
	dw  - disabled write to the mapped file
	lo  - pages are locked in memory
	io  - memory mapped I/O area
	sr  - sequential read advise provided
	rr  - random read advise provided
	dc  - do not copy area on fork
	de  - do not expand area on remapping
	ac  - area is accountable
	nr  - swap space is not reserved for the area
	ht  - area uses huge tlb pages
	nl  - non-linear mapping
	ar  - architecture specific flag
	dd  - do not include area into core dump
	sd  - soft-dirty flag
	mm  - mixed map area
	hg  - huge page advise flag
	nh  - no-huge page advise flag
	mg  - mergable advise flag
	

>当swap报警时，处理方法

	#使用iotop
	yum install iotop

	#基于smap内容，查看应用swap占用率
	for i in $( cd /proc;ls |grep "^[0-9]"|awk ' $0 >100') ;do awk '/Swap:/{a=a+$2}END{print '"$i"',a/1024"M"}' /proc/$i/smaps 2>/dev/null ; done | sort -k2nr | head

	#强行清除交换分区
	swapoff -a && swapon -a