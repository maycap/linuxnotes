###hadoop###
***

###前言###

简单记录下hadoop搭建步骤，具体见官网

***

####部署hadoop.tar.gz
	#查找列表 http://hadoop.apache.org/releases.html
	wget http://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz
	
	#解压，配置环境变量
	tar -zxvf hadoop-2.6.0.tar.gz
	
	vim /etc/profile
	export HADOOP_HOME=/web/hadoop-2.6.0
	export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin  
	#H-Master可以多加一项，便于操作
	export HADOOP_HOME_WARN_SUPPESS=1

####添加jdk1.7
	
	vim /etc/profile
	export JAVA_HOME=/web/jdk1.7.0_67
	export JRE_HOME=$JAVA_HOME/jre
	export CLASSPATH=.:$CALSSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
	export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

####添加hosts
	vim /etc/hosts
	192.168.100.100 HMaster
	192.168.100.101 HSlave1
	192.168.100.102 HSlave2
	192.168.100.103 HSlave3
	192.168.100.104 HClient

####配置hadoop
hadoop默认配置文件在hadoop-2.6/etc/hadoop/下

* 在*-env.sh中加入java的环境变量

	export JAVA_HOME=/web/jdk1.7.0_67

* 修改*-site.xml的配置文件

>cat core-site.xml

	<configuration>
    	<property>
        	<name>fs.defaultFS</name>
        	<value>hdfs://HMaster:9100</value>
    	</property>
    	<property>
        	<name>hadoop.tmp.dir</name>
        	<value>/web/hadoop_data/tmp</value>
    	</property>
	</configuration>

	
特别注意：如没有配置hadoop.tmp.dir参数，此时系统默认的临时目录为：/tmp/hadoo-hadoop。而这个目录在每次重启后都会被删除，必须重新执行format才行，否则会出错。

>cat hdfs-site.xml

	<configuration>
		<property>
	        <name>dfs.replication</name>
        	<value>3</value>
    	</property>
    	<property>
	        <name>dfs.namenode.name.dir</name>
	        <value>/web/hadoop_data/hdfs/name</value>
	    </property>
	    <property>
	        <name>dfs.datanode.data.dir</name>
	        <value>/web/hadoop_data/hdfs/data</value>
    	</property>
    	<property>
        	<name>dfs.nameservices</name>
        	<value>hadoop-cluster1</value>
	    </property>
	    <property>
	        <name>dfs.namenode.secondary.http-address</name>
	        <value>HMaster:50090</value>
	    </property>
	    <property>
	        <name>dfs.webhdfs.enabled</name>
	        <value>true</value>
	    </property>
	</configuration>
	
>cat mapred-site.xml

	<configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
            <final>true</final>
        </property>
	    <property>
	        <name>mapreduce.jobtracker.http.address</name>
	        <value>HMaster:50030</value>
	    </property>
	    <property>
	        <name>mapreduce.jobhistory.address</name>
	        <value>HMaster:10020</value>
	    </property>
	    <property>
	        <name>mapreduce.jobhistory.webapp.address</name>
	        <value>HMaster:19888</value>
	    </property>
        <property>
            <name>mapred.job.tracker</name>
            <value>http://HMaster:9001</value>
        </property>
	</configuration>

>cat yarn-site.xml

	<configuration>
	
        <!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>HMaster</value>
        </property>
	
	    <property>
	        <name>yarn.nodemanager.aux-services</name>
	        <value>mapreduce_shuffle</value>
	    </property>
	    <property>
	        <name>yarn.resourcemanager.address</name>
	        <value>HMaster:8032</value>
	    </property>
	    <property>
	        <name>yarn.resourcemanager.scheduler.address</name>
	        <value>HMaster:8030</value>
	    </property>
	    <property>
	        <name>yarn.resourcemanager.resource-tracker.address</name>
	        <value>HMaster:8031</value>
	    </property>
	    <property>
	        <name>yarn.resourcemanager.admin.address</name>
	        <value>HMaster:8033</value>
	    </property>
	    <property>
	        <name>yarn.resourcemanager.webapp.address</name>
	        <value>HMaster:8088</value>
	    </property>
	</configuration>

####ssh通信####
	ssh-keygen -t rsa
    ssh-copy-id -i id_rsa.pub HMaster(HSlave1...）
	使H-Master和H-Slaves可以通信	


####分派masters,slaves

		cat masters   ---通用配置
			H-Master
	    cat slaves      ---master专属配置
			H-Slave1
			H-Slave2
			....

####初始化HDFS文件系统，仅需要一次
	 hadoop namenode -format

####启动hadoop
	start-all.sh

####检测手段jps
	H-Master：
	4448 ResourceManager
	8704 Jps
	4073 NameNode
	4257 SecondaryNameNode

	H-Slave:
	5419 Jps
	5340 NodeManager
	1740 DataNode

####HClient使用
	复制HMaster使用的hadoop到HClient所在的机器，配置同，不用启动

***
####友情提示
默认编译时32位，64位系统使用，自主编译hadoop，替换lib/native即可
	
	


