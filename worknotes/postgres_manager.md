##postgres管理##

###前言##
postgres数据库管理笔记

***
####数据库安装

>必须包

	yum install gcc readline-devel zlib-devel

>源码编译

	
	groupadd postgres
	useradd -g postgres postgres

	tar -xf postgresql-9.2.3..tar.gz
	cd postgresql-9.2.3
	./configure --prefix=/web/pgsql
	make
	make install

	mkdir /web/data
	chown -R postgres.postgres /web/data
	su - postgres
	/web/pgsql/bin/initdb -D /web/data


####授权管理

	#创建用户
	CREATE USER postgresuser WITH PASSWORD 'xxxx'; 

	#修改密码
	alter user postgresuser with password 'new password';

	#进入对应库授权
	GRANT select ON ALL TABLES IN SCHEMA public TO postgresuser;

####慢查询处理

数据库节点io,负载飙升都是常事，sql检测方法有：

>版本9.1

	SELECT procpid, datname, xact_start, query_start, current_query, waiting
	FROM pg_stat_activity 
	where current_query <> '<IDLE>'  
	and extract(epoch from CURRENT_TIMESTAMP - query_start) >= 0
	order by query_start;

>版本9.2+

	SELECT datname, pid, client_addr, backend_start, xact_start, query_start, state_change, waiting, state, query
	FROM pg_stat_activity 
	where state <> 'idle' 
	order by xact_start; 

sql回滚和强杀：

	select pg_cancel_backend('pid'); 
	select pg_terminate_backend('pid');

####表磁盘使用量判断

每个表都有一个主堆(primary heap)磁盘文件，大多数数据都存储在这里。 如果一个表存在值可能会很长的字段，则另外还有一个用于存储因为数值 太长而不适合存储在主表中的数据的TOAST文件。如果存在这个扩展表，那么将会同时存在 一个TOAST索引。当然，同时还可能有索引和基表关联。每个表和索引都存放在单独的磁盘文件里(超过 1GB 可能会被分割成多个)。

>检测某种类表占用大小

	SELECT
		pg_relation_filepath (oid),
		relname,
		relpages
	FROM
		pg_class
	WHERE
		relname LIKE '%uc%'
	ORDER BY
		relpages DESC;

输出结果如：

<table>
	<tr>
		<td>pg_relation_filepath</td>
		<td>relname</td>
		<td>relpages</td>
	</tr>
	<tr>
		<td>base/16384/178210</td>
		<td>t_uc_organize_rel</td>
		<td>9882</td>
	</tr>
</table>	

字段说明：

	pg_relation_filepath	空间数据真实存在路径(父目录：pgdata）
	relname				    表名
	relpages				表所占页数（8kb/page)

	du -sh $pg_relation_filepath =  $relpages * 8     

>显示表索引尺寸，一张表可能有很多索引
	
	SELECT
		c2.relname,
		c2.relpages
	FROM
		pg_class C,
		pg_class c2,
		pg_index i
	WHERE
		C .relname = 't_uc_user'
	AND C .oid = i.indrelid
	AND c2.oid = i.indexrelid
	ORDER BY
		c2.relname;


####修改备份及恢复
对生成环境修改数据，可能要回滚回去，备份修改的数据是必要的，简单方法如下：

1.建立schema

	create schema bak;

2.将delete,update语句修改为select，查询插入新表

	CREATE TABLE "bak".uc_group_20150814 AS (SELECT * FROM uc_group WHERE app_code='mytest');

3.恢复

	#要求目标表target_table不存在，因为在插入时会自动创建	
	select * into target_table from source_table;

	#要求目标表target_table存在，由于目标表已经存在，所以我们除了插入源表source_table的字段外，还可以插入常量，如例中的：5。
	insert into target_table(column1,column2) select column1,5 from source_table; 

	#命令恢复
	pg_dump uc -t uc_group_20150814 -a > psql uc -t uc_group	


####模拟内存表

1.建立tmpfs（ramfs无法限制使用大小）
	
	mkdir /web/tmpfs_8G
	mount -t tmpfs -o size=8G tmpfs /web/tmpfs_8G
	chown -R postgres.postgres /web/tmpfs_8G

2.建立表空间

	psql> create tablespace tmpfs location '/web/tmpfs_8G';

3.使用表空间

	psql> create database tmpfsdb template template0 owner postgres tablespace tmpfs;


####垃圾收集与分析

>Name

	VACUUM -- 垃圾收集以及可选地分析一个数据库

>Synopsis

	VACUUM [ FULL | FREEZE ] [ VERBOSE ] [ table ]
	VACUUM [ FULL | FREEZE ] [ VERBOSE ] ANALYZE [ table [ (column [, ...] ) ] ]

>参数
	
	FULL
	选择"完全"清理，这样可以恢复更多的空间， 但是花的时间更多并且在表上施加了排它锁。
	
	FREEZE
	选择激进的元组"冻结"。
	
	VERBOSE
	为每个表打印一份详细的清理工作报告。
	
	ANALYZE
	更新用于优化器的统计信息，以决定执行查询的最有效方法。
	
	table
	要清理的表的名称（可以有模式修饰）。缺省时是当前数据库中的所有表。
	
	column
	要分析的具体的列/字段名称。缺省是所有列/字段。
