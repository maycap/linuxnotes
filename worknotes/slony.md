##slony##

###前言###
基于表备份的数据库同步，slony很是优秀，整理笔记。

***

###传统配置###

slony有点类似postgresql库的概念，一个完整的slony复制称为一个集群，在schema下表现为_$CLUSTER。通过内部表信息，可以看到执行到哪一步，以及是否缺少信息，执行过程的日志自然更方便。集群下可以有无数个集合，slony复制以set作为一个集，集内部可以定义数个需要同步的表信息，前提需要ddl必须相同！同步开始前，从库会被清空，因此无须复制数据到从库！

>init_uc.sh   --初始化信息，仅仅需要一次即可

	#!/bin/sh

	#slonik可执行文件位置
	SLONIK=/web/pgsql/bin/slonik
	
	#你的集群的名称
	CLUSTER=uc
	
	#你的复制集的名称
	SET_ID=1
	
	#主服务器ID
	MASTER=1
	
	#源库IP或主机名
	MASTER_HOST=cloud2
	
	#需要复制的源数据库
	MASTER_DBNAME=uc
	
	#源库数据库超级用户名
	MASTER_USER=postgres
	
	#源库数据库超级用户密码
	MASTER_PASSWORD=mypassword
	
	#源库数据库端口
	MASTER_PORT=5432
	
	#从服务器ID
	SLAVE1=2
	
	#目的库IP或主机名
	SLAVE1_HOST=cloud2
	
	#需要复制的目的数据库
	SLAVE1_DBNAME=report
	
	#目的库用户名
	SLAVE1_USER=postgres
	
	#目的库用户密码
	SLAVE1_PASSWORD=mypassword
	
	#目的库数据库端口
	SLAVE1_PORT=5433
	
	$SLONIK <<_EOF_
	
	#这句是定义集群名
	cluster name = $CLUSTER;
	
	#这两句是定义复制节点
	node $MASTER admin conninfo = 'dbname=$MASTER_DBNAME host=$MASTER_HOST user=$MASTER_USER password=$MASTER_PASSWORD port=$MASTER_PORT';
	node $SLAVE1 admin conninfo = 'dbname=$SLAVE1_DBNAME host=$SLAVE1_HOST user=$SLAVE1_USER password=$SLAVE1_PASSWORD port=$SLAVE1_PORT';
	
	#初始化集群和主节点，id从1开始，如果只有一个集群，那么肯定是1
	#comment里可以写一些自己的注释，随意
	init cluster ( id = $MASTER, comment = 'Primary Node' );
	
	#下面是从节点
	store node ( id = $SLAVE1, comment = 'Slave1 Node', event node=$MASTER );
	
	#配置主从两个节点的连接信息，就是告诉Slave服务器如何来访问Master服务器
	#下面是主节点的连接参数
	store path ( server = $MASTER, client = $SLAVE1,conninfo = 'dbname=$MASTER_DBNAME host=$MASTER_HOST user=$MASTER_USER password=$MASTER_PASSWORD port=$MASTER_PORT');
	
	#下面是从节点的连接参数
	store path ( server = $SLAVE1, client = $MASTER,conninfo = 'dbname=$SLAVE1_DBNAME host=$SLAVE1_HOST user=$SLAVE1_USER password=$SLAVE1_PASSWORD port=$SLAVE1_PORT');
	
	#设置复制中角色，主节点是原始提供者，从节点是接受者
	store listen ( origin = $MASTER, provider = $MASTER, receiver = $SLAVE1 );
	store listen ( origin = $SLAVE1, provider = $SLAVE1, receiver = $MASTER );
	
	#创建一个复制集，id也是从1开始
	create set ( id = $SET_ID, origin = $MASTER, comment = 'uc Database All tables' );
	
	#向自己的复制集种添加表，每个需要复制的表添加一条set命令，id从1开始，逐次递加，步进为1；
	#fully qualified name是表的全称：模式名.表名
	#这里的复制集id需要和前面创建的复制集id一致
	set add table ( set id = $SET_ID, origin = $MASTER,id = 1,  fully qualified name = 'public.t_uc_group',comment = 'Table t_uc_group' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 2,  fully qualified name = 'public.t_uc_login',comment = 'Table t_uc_login' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 3,  fully qualified name = 'public.t_uc_user',comment = 'Table t_uc_user' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 4,  fully qualified name = 'public.t_uc_login_log',comment = 'Table t_uc_login_log' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 5,  fully qualified name = 'public.t_uc_group_category',comment = 'Table t_uc_group_category' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 6,  fully qualified name = 'public.t_uc_organize_rel',comment = 'Table t_uc_organize_rel' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 7,  fully qualified name = 'public.t_uc_group_member',comment = 'Table t_uc_group_member' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 8,  fully qualified name = 'public.t_uc_organize',comment = 'Table t_uc_organize' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 9,  fully qualified name = 'public.t_uc_position',comment = 'Table t_uc_position' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 10,  fully qualified name = 'public.t_uc_user_detail',comment = 'Table t_uc_user_detail' );
	set add table ( set id = $SET_ID, origin = $MASTER,id = 11,  fully qualified name = 'public.t_uc_user_gen_group_role',comment = 'Table t_uc_user_gen_group_role' );
	
	#提交复制集
	subscribe set ( id = $SET_ID, provider = $MASTER, receiver = $SLAVE1, forward = yes);
	_EOF_
	########################

	
为了便于管理，启动脚本信息单独抽出来:

>库信息

	# cat master_uc.slon
	########################
	cluster_name=uc
	conn_info="dbname=uc host=cloud2 user=postgres password=mypassword port=5432"
	########################

	# cat slave_uc.slon
	########################
	cluster_name=uc
	conn_info="dbname=report host=cloud2 user=postgres password=mypassword port=5433"
	########################


>启动脚本

	# cat start_master.sh 
	##uc
	/web/pgsql/bin/slon -p /web/slony/report/uc/master_uc.slon.pid -f /web/slony/report/uc/master_uc.slon >>/web/slony/report/uc/master_uc.slon.log &

	#cat start_slave.sh
	/web/pgsql/bin/slon -p /web/slony/report/uc/slave_uc.slon.pid -f /web/slony/report/uc/slave_uc.slon >>/web/slony/report/uc/slave_uc.slon.log &

>关闭脚本

	#cat stop_master.sh
	##uc
	kill -quit `cat /web/slony/report/uc/master_uc.slon.pid`
	rm -rf /web/slony/report/uc/master_uc.slon.pid
	rm -rf /web/slony/report/uc/master_uc.slon.log

	#cat stop_slave.sh
	##uc
	kill -quit `cat /web/slony/report/uc/slave_uc.slon.pid`
	rm -rf /web/slony/report/uc/slave_uc.slon.pid
	rm -rf /web/slony/report/uc/slave_uc.slon.log

>添加新表，slony向原先的set添不进去，"ERROR:  Slony-I: cannot add table to currently subscribed set 1 - must attach to an unsubscribed set" ，需要create set

	#!/bin/sh
	#drop trigger  _els_denyaccess on t_els_study_delay
	/web/pgsql/bin/slonik<<_EOF_
	cluster name = els;
	node 1 admin conninfo = 'dbname=std1 host=cloud2 user=postgres password=mypassword port=5433';
	node 2 admin conninfo = 'dbname=report_std1 host=cloud2 user=postgres password=mypassword port=5433';
	create set ( id = 3,origin=1,comment='add els set');
	set add table ( set id = 3,origin = 1,id=22,fully qualified name = 'public.t_els_study_delay',comment='add new table');
	subscribe set (id=3,provider=1,receiver=2);
	_EOF_


###脚本配置###

在初始完集群的情况下，曾经写了一个，自动建库，自动建立slony基于全库表配置的脚本。可自定义setid,tableid，schemaname,以及指定配置文件，基于一主多从，从库已逗号分隔。

具体内容见脚本: [init_schema](../giftscript/init_schema)

	[postgres@v4app18 ~]$ ./init_schema -h
	init_schema is the config slony script ,copy schema and auto config.
	
	Usage:
		init_schema [OPTION]... 
		Example: -n [cluster] -f [sql] -S [schemaname] -m "comment" 
	
	General options:
		-h request for help.
		-n select cluster name.
		-S new schema name.
		-f select sql.
		-t new setid.
		-T new table id.
		-m this is comment.
		-c select your configure file(default ./cluster.config).
		-v look version.
		-s syn (0|all,1|master,2|slave) DDL.

	[postgres@v4app18 ~]$ ./init_schema -n gem -f dsc_public.sql -S maycap -m "my test"
	Now create schema on 192.168.1.218 9432 dsc...
	192.168.1.218:9432:dsc is ok!
	Now create schema on 192.168.1.201 9434 cap...
	192.168.1.201:9434:cap is ok!
	Now config slony....
	Slony Cluster gem setid will be 3.
	New add table will be 16.
	Now init new set...
	All Done!

	#脚本读取的默认的配置文件
	[postgres@v4app18 ~]$ cat cluster.config 
	gem_masterhost:192.168.1.218
	gem_masterport:9432
	gem_masterdbname:dsc
	gem_masternode:1
	gem_slavehost:192.168.1.201
	gem_slaveport:9434
	gem_slavedbname:cap
	gem_slavenode:2


###问题###

>slony正在运行时，删除修改表失败

	PGRES_FATAL_ERROR ERROR:  Slony-I: setAddTable_int(): table "public"."t_els_course_study_record" has no index t_els_course_study_record_pkey

	
	#查看对应表结构
	CONSTRAINT "pk_t_els_exam_user" PRIMARY KEY ("exam_user_id")

	CONSTRAINT "t_els_exam_user_pkey" PRIMARY KEY ("exam_user_id")

应先停掉slony，在新建表，然后在删除原先触发器

	_els_logtrigger
	_els_truncatedeny
	_els_truncatetrigger
	_els_denyaccess

>更新表脚本

	#!/bin/sh
	
	for i in `cat uc.list`
	do
		echo "Now dump $i"
		psql -U postgres  -p 5433 std1 -c "drop trigger _uc_logtrigger on $i"
		psql -U postgres  -p 5433 std1 -c "drop trigger _uc_truncatedeny on $i"
		psql -U postgres  -p 5433 std1 -c "drop trigger _uc_truncatetrigger on $i"
		psql -U postgres  -p 5433 std1 -c "drop trigger _uc_denyaccess on $i"
		pg_dump -U postgres  -p 5433 -t $i std1 -s > $i.sql
		echo "同步report---$i"
		psql -U postgres -p 5433 -d report_std1 -c "drop table $i"
		psql -U postgres -p 5433 -d report_std1 -f ./$i.sql
		rm -fr ./$i.sql
	done

