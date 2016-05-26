##postgresql hot_standby

###前言
PostgreSQL hot standby就是实现多个PostgreSQL节点实现数据同步(其实9.0x不只是异步)、这个同步是针对整数集群的(包含一切的数据、 DDL,DCL都会在salve上同步)。salve在利用日志恢复数据同时也能提供只读的操作，这样就可以利用这个技术实现多台主机数据同步和读取操作 的负载平衡。

###部署

####主库

>初始化库

	mkdir /web/data	
	chown -R postgres.postgres /web/data
	
>postgresql.conf必要配置项

	listen_addresses = '*'
	wal_level = hot_standby
	synchronous_commit = on
	max_wal_senders = 2
	wal_keep_segments = 32
	synchronous_standby_names = '*'

>pg_hba.conf必要配置项

	#允许局域网中md5密码认证连接
	host    all             all        192.168.100.0/24        md5
	#用户数据同步，必须为replication数据库
	host  replication       repl          192.168.100.0/24        trust

>添加用户

	su - postgres  pg_ctl -D /web/data start
	psql -U postgres -p 5432
	create user repl with replication password 'repl.repl';

>开启热备功能

	psql -U postgres -p 5432
	>select pg_start_backup('mylable');
	>select pg_stop_backup();
	
>传输

	su - postgres pg_ctl -D /web/data stop
	scp -r /web/data  SLAVE:/web

####从库

>postgresql.conf修改配置项

	wal_level = minimal
	synchronous_commit = off
	max_wal_senders = 0
	wal_keep_segments = 0
	synchronous_standby_names = ''
	hot_standby = on

>获取recovery.conf文件

	cp  /opt/postgresql/share/recovery.conf.sample  /web/data

>编译recovery.conf文件

	standby_mode = 'on'
	primary_conninfo = 'host=MASTER port=5432 user=repl password=repl.repl'
	trigger_file = '/web/data/trigger_activestb'	

>启动主库，在启动从库
	
###检测

>新建库表数据检测法

	在主库新建库，表，插入数据，检测从库是否有对应数据出现。

>ps 查看法

	ps -ef | grep wal

	--wal  sender 是主库标志
	postgres 18147 18139  0 19:32 ?        00:00:00 postgres: wal writer process             
	postgres 18171 18139  0 19:32 ?        00:00:00 postgres: wal sender process repl 192.168.x.x(59648) streaming 0/79F98F8

	--wal receiver 是从库标志
	postgres   318   312  0 11:30 ?        00:00:01 postgres: wal receiver process   streaming 0/3021480

> pg_controldata  /web/data

	-- in production 是主库标志
	pg_control version number:            942
	Catalog version number:               201510051
	Database system identifier:           6286792961765534492
	Database cluster state:               in production

	-- in archive recovery 是从库标志
	pg_control version number:            942
	Catalog version number:               201510051
	Database system identifier:           6286792961765534492
	Database cluster state:               in archive recovery



	

	



