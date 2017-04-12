## Tomcat

### 前言

Apache Tomcat是一款实现了 Java Servlet, JavaServer Pages, Java Expression Language and Java WebSocket technologies的开源软件，属于轻量级应用服务器。

### Tomcat内容介绍

环境使用Centos6.5，参考官网文档: [http://tomcat.apache.org/](http://tomcat.apache.org/)

>Apache Tomcat Versions

<table>
	<tr>
		<td>Servlet Spec</td>
		<td>JSP Spec</td>
		<td>EL Spec</td>
		<td>WebSocket Spec</td>
		<td>Apache Tomcat version</td>
		<td>Actual release revision</td>
		<td>Support Java Versions</td>		
	</tr>
	<tr>
		<td>3.1</td>
		<td>2.3</td>
		<td>3.0</td>
		<td>1.1</td>
		<td>8.0.x</td>
		<td>8.0.24</td>
		<td>7 and late</td>		
	</tr>
		<tr>
		<td>3.0</td>
		<td>2.2</td>
		<td>2.2</td>
		<td>1.1</td>
		<td>7.0.x</td>
		<td>7.0.63</td>
		<td>6 and later
(WebSocket 1.1 requires 7 or later)</td>		
	</tr>
	<tr>
		<td>2.5</td>
		<td>2.1</td>
		<td>2.1</td>
		<td>N/A</td>
		<td>6.0.x</td>
		<td>6.0.24</td>
		<td>5 and late</td>		
	</tr>
</table>


>Contents列表


<table>
	<tr>
		<td>位置</td>
		<td>描述</td>	
	</tr>
	<tr>
		<td>bin/</td>
		<td>启动，关闭，JVM配置等相关项目录</td>
	</tr>
	<tr>
		<td>conf/</td>
		<td>配置文件和相关的DTD。在这里最重要的文件是server.xml中。它是为容器的主配置文件。</td>
	</tr>
	<tr>
		<td>lib/</td>
		<td>tomcat自身依赖的lib包所在目录</td>
	</tr>
	<tr>
		<td>LICENSE</td>
		<td>许可证文件</td>
	</tr>
	<tr>
		<td>logs/</td>
		<td>默认存放日志的目录</td>
	</tr>
	<tr>
		<td>NOTICE</td>
		<td>许可证信息和例外</td>
	</tr>
	<tr>
		<td>RELEASE-NOTES</td>
		<td>发行版信息</td>
	</tr>
	<tr>
		<td>RUNNING.txt</td>
		<td>如何开始的使用说明文档</td>
	</tr>
	<tr>
		<td>temp/</td>
		<td>临时文件目录</td>
	</tr>
	<tr>
		<td>webapps</td>
		<td>默认配置运行webapps的目录</td>
	</tr>
	<tr>
		<td>work</td>
		<td>把jsp转换为class文件的工作目录，定时刷新非及时刷新</td>
	</tr>
</table>

>catalina.home与catalina.base区别

catalina.home
	
此选项主要定义tomcat发行版自身一些属性,即公用信息位置，就是bin/,lib/目录。

catalina.base

此选项主要定义指定的tomcat服务实例相关位置，如：conf、logs、temp、webapps和work。

仅运行一个Tomcat实例时，这两个属性指向的位置是相同的。

### 插一点自带功能介绍

将tomcat单独作为web服务器，功能将复杂很多。常使用作为后端容器，再此少量介绍。直接运行，打开对应的 http://ip:8080 ,页面功能，使用较少，主要为单人测试使用。

>Server Status ：主要展示JVM占用和链接信息等

1.访问控制文件为$CATALINA_HOME/conf/tomcat-users.xml

	<role rolename="tomcat"/>
	<role rolename="manager-gui"/>
	<user username="tomcat" password="tomcat" roles="tomcat,manager-gui"/>

2.若是只保留上述目录，直接访问路径为 

	http://ip:8080/manager/status

>Manager App :提供管理应用的功能

1.应用start、stop、reload、Undeploy功能

2.新增应用功能

>host manager :虚拟主机管理

1.访问控制文件为$CATALINA_HOME/conf/tomcat-users.xml

	<role rolename="admin-gui"/>
	<user username="tomcat" password="tomcat" roles="tomcat,manager-gui，admin-gui"/>


### 配置介绍

>丢war方式

webapps是默认的运行web应用实例的目录，也是常见使用官方war包最简单的方式。若tomcat本身是启动的，丢进去会自动解压运行。

>虚拟目录方式

从官网新下的tomcat默认没有，只要运行一下，会自动创建一个目录为：$CATALINA_BASE/conf/Catalina/localhost/。采用XML配置文件格式，可简单配置虚拟应用如下：

	！！友情提示:
	webapp访问的虚拟目录取决于：项目自身设置的访问目录层级，一个项目在编译后，项目访问名已固定。
	一般项目打包名就是虚拟目录名，则配置文件可归为下：

	#访问方式 http://127.0.0.1:8080/myappname

	#cat  myappname.xml
	<Context path="/myappname" docBase="/web/myappname" >
	</Context>

	==》为了减少非必要问题，设置一样比较简单易管理。


如果项目需要直接访问，不加虚拟目录

	--先修改$CATALINA_BASE/conf/web.xml
    <init-param>
        <param-name>listings</param-name>
        <param-value>false</param-value>    -->true
    </init-param>

	#cat  mytest.xml
	<Context path="" docBase="/web/mytest" >
	</Context>

>上述失效，走下述
	修改 Tomcat/conf/server.xml文件

	<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true"
            xmlValidation="false" xmlNamespaceAware="false">

	--添加如下一行即可
        <Context path="" docBase="/web/yourapp" reloadable="true" />
	


>端口修改配置

1.首先修改默认端口，主文件conf/server.xml

	#截取端口配置条目

	--远程停止服务端口
	<Server port="8005" shutdown="SHUTDOWN">

	--8080是http端口，8443是https端口
    <Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />

	--AJP端口，APACHE能过AJP协议访问TOMCAT的8009端口
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

若是一台机器部署一堆tomcat，随意修改端口，恐怕服务很难起得来。采用前同事传下来的经验，以myapp应用为例，指定端口为8001，则一系列端口修改可为：

	SHUTDOWN port   =>  8401
	http port		=>	8001
	https port		=>	8601
	AJP port		=>	8801

>开启调试端口

生成环境问题，线下重现，断点调试，是个常见的过程，tomcat开启断点功能是很必须的。主配置文件为$CATALINA_BASE/bin/catalina.sh

	--catalina.sh中关于调试断口相关介绍
	JPDA_ADDRESS    (Optional) Java runtime options used when the "jpda start"
    command is executed. The default is localhost:8000.

	--catalina.sh中关于添加变量提示
	Do not set the variables in this script. Instead put them into a script
	setenv.sh in CATALINA_BASE/bin to keep your customizations separate.

==>步骤如下：

	--添加文件
	touch CATALINA_BASE/bin/setenv.sh

	--定义调试端口
	echo "JPDA_ADDRESS=8201" >>setenv.sh

	--在启动脚本中添加 jpda start
	vim CATALINA_BASE/bin/startup.sh
	exec "$PRGDIR"/"$EXECUTABLE" jpda start "$@"

>设置JVM

内存是最容易出问题的地方，JVM设置下是必要的。基于上文catalina.sh的友情建议，写在setenv.sh里，参考如下：

	export JAVA_OPTS="$JAVA_OPTS -Duser.timezone=GMT+8  \
	-server -Xms512m -Xmx512m -XX:NewSize=256m -XX:MaxNewSize=256m \
	-XX:PermSize=128m -XX:MaxPermSize=256m \
	-XX:SurvivorRatio=16 -XX:MaxTenuringThreshold=5 \
	-XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection \
	-XX:CMSMaxAbortablePrecleanTime=500 -XX:+CMSClassUnloadingEnabled \
	-verbose.gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps \
	-Xloggc:$CATALINA_HOME/logs/gc.log -Djava.awt.headless=true"


>多JDK环境

jdk版本整体改动是很困难的，但是少数项目可能使用新版jdk，因此承载新webapp项目的tomcat可能需要指定jdk版本，而不是读取环境变量中的。参考上述JPDA_ADDRESS，依然在setenv.sh中

	
	--catalina.sh友情提示
	JAVA_HOME      Must point at your Java Development Kit installation.
                   Required to run the with the "debug" argument.
	
	JRE_HOME       Must point at your Java Runtime installation.
	               Defaults to JAVA_HOME if empty. If JRE_HOME and JAVA_HOME
	               are both set, JRE_HOME is used.
	

	Ensure that any user defined CLASSPATH variables are not used on startup,
	but allow them to be specified in setenv.sh, in rare case when it is needed.
	CLASSPATH=
	
	if [ -r "$CATALINA_BASE/bin/setenv.sh" ]; then
	  . "$CATALINA_BASE/bin/setenv.sh"
	elif [ -r "$CATALINA_HOME/bin/setenv.sh" ]; then
	  . "$CATALINA_HOME/bin/setenv.sh"
	fi

因此在setenv.sh设置，catalina.sh会读取加载，参考配置：

	export JAVA_HOME=/web/jdk1.7.0_67
	export JRE_HOME=/web/jdk1.7.0_67/jre
	export CLASSPATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib
	
>连接数设置，在server.xml中

1.打开共享的线程池:

	--默认是注释掉，去掉注释即可	
	<Service name="Catalina">
	
	<!--The connectors can use a shared executor, you can define one or more named thread pools-->
	<!--
		<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
		maxThreads="150" minSpareThreads="4" maxIdleTime="600000"/>
	-->

	--属性值介绍：
	name：共享线程池的名字。这是Connector为了共享线程池要引用的名字，该名字必须唯一。
	namePrefix:在JVM上，每个运行线程都可以有一个name 字符串。这一属性为线程池中每个线程的name字符串设置了一个前缀，Tomcat将把线程号追加到这一前缀的后面。
	maxThreads：该线程池可以容纳的最大线程数。；
	maxIdleTime：在Tomcat关闭一个空闲线程之前，允许空闲线程持续的时间(以毫秒为单位)。只有当前活跃的线程数大于minSpareThread的值，才会关闭空闲线程。
	minSpareThreads：Tomcat应该始终打开的最小不活跃线程数。
	threadPriority：线程的等级。默认是Thread.NORM_PRIORITY。

2.在Connector中指定使用共享线程池：

	--手动加入
	<Connector executor="tomcatThreadPool"
           port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" 
           minProcessors="5"
           maxProcessors="75"
           acceptCount="1000"/>
 
	--属性值介绍：
	executor：表示使用该参数值对应的线程池；
	minProcessors：服务器启动时创建的处理请求的线程数；
	maxProcessors：最大可以创建的处理请求的线程数；
	acceptCount：指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。	
	

