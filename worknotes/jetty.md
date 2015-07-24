##jetty

###前言###

Jetty是一个开源的项目，可以提供http服务端，客户端和javax.servlet服务的容器。下载地址：[http://download.eclipse.org/jetty/](http://download.eclipse.org/jetty/) 

###Jetty内容介绍

>jetty Versions


<table>
	<tr>
		<td>Version</td>
		<td>JVM</td>
		<td>Protocols</td>
		<td>Servlet</td>
		<td>JSP</td>
		<td>Status</td>		
	</tr>
	<tr>
		<td>9.3</td>
		<td>1.8</td>
		<td>HTTP/1.1 (RFC 7230), HTTP/2 (RFC 7540), WebSocket (RFC 6455, JSR 356), FastCGI</td>
		<td>3.1</td>
		<td>2.3</td>
		<td>Stable</td>		
	</tr>
	<tr>
		<td>9.2</td>
		<td>1.7</td>
		<td>HTTP/1.1 RFC2616,javax.websocket, SPDY v3</td>
		<td>3.1</td>
		<td>2.3</td>
		<td>Stable</td>		
	</tr>
	<tr>
		<td>8</td>
		<td>1.6</td>
		<td>HTTP/1.1 RFC2616, WebSocket RFC 6455, SPDY v3</td>
		<td>3.0</td>
		<td>2.2</td>
		<td>Mature</td>		
	</tr>
	<tr>
		<td>7</td>
		<td>1.5</td>
		<td>HTTP/1.1 RFC2616, WebSocket RFC 6455, SPDY v3</td>
		<td>2.5</td>
		<td>2.1</td>
		<td>Mature</td>		
	</tr>
</table>


>Contents列表


<table>
	<tr>
		<td>位置</td>
		<td>描述</td>	
	</tr>
	<tr>
		<td>license-eplv10-aslv20.html</td>
		<td>Jetty的许可证文件</td>
	</tr>
	<tr>
		<td>README.txt</td>
		<td>如何开始的使用说明文档</td>
	</tr>
	<tr>
		<td>VERSION.txt</td>
		<td>发行版信息</td>
	</tr>
	<tr>
		<td>bin/</td>
		<td>运行Jetty的shell脚本所在目录</td>
	</tr>
	<tr>
		<td>demo-base/</td>
		<td>Jetty基础目录用来运行Jetty的演示程序</td>
	</tr>
	<tr>
		<td>etc/</td>
		<td>Jetty的xml配置目录</td>
	<tr>
		<td>lib/</td>
		<td>运行Jetty的所有必须JAR所在目录</td>
	</tr>
	<tr>
		<td>logs/</td>
		<td>请求日志目录</td>
	</tr>
	<tr>
		<td>modules/</td>
		<td>模块定义目录</td>
	</tr>
	<tr>
		<td>notice.html</td>
		<td>许可证信息和例外</td>
	</tr>
	<tr>
		<td>resources/</td>
		<td>配置扩展目录，需要激活包含此目录</td>
	</tr>	
	<tr>
		<td>notice.html</td>
		<td>许可证信息和例外</td>
	</tr>
	<tr>
		<td>start.d/</td>
		<td>start.ini扩展目录</td>
	</tr>
	<tr>
		<td>start.ini</td>
		<td>添加命令行参数如：模块，属性，XML配置文件</td>
	</tr>
	<tr>
		<td>start.jar</td>
		<td>jar启动Jetty</td>
	</tr>
	<tr>
		<td>notice</td>
		<td>Jetty默认配置运行webapps的目录</td>
	</tr>
</table>

>jetty.home与jetty.base区别

jetty.home
	
此选项主要定义jetty发行版自身一些属性，比如自身的libs，默认的模块，默认的XML配置文件（如start.jar,lib,etc）

jetty.base

此选项主要定义指定的jetty服务实例位置，配置文件，日志和web应用（如start.ini,start.d,logs and webapps)

>启动演示案例小插曲：

Jetty版本对应JVM版本，演示案例则需要对应的java版本启动

	Exception in thread "main" java.lang.UnsupportedClassVersionError: org/eclipse/jetty/start/Main : Unsupported major.minor version 52.0

	小直觉： 52.0 <=> jdk1.8  |  51.0 <=> jdk1.7


>jetty直接启动应用方式

	cd $JETTY_BASE

	#指定端口，默认8080
	java -jar $JETTY_HOME/start.jar jetty.http.port=8081

	#Adding SSL for HTTPS & HTTP2
	java -jar $JETTY_HOME/start.jar --add-to-startd=https,http2
	
	#Changing the Jetty HTTPS Port
	java -jar $JETTY_HOME/start.jar jetty.ssl.port=8444

	#有用的help
	java -jar $JETTY_HOME/start.jar --help

>修改start.ini，指定配置参数，与后缀--等价，更方便些

	## HTTP port to listen on
	jetty.port=8081

	----这个是读取当前目录下的start.ini，比如我们在Jetty目录修订为8081。
		进入demo-base，运行jetty,端口已然是演示实例下start.ini的设置。

>添加虚拟应用

1.根据应用jdk配置$JETTY_HOME/bin/jetty.sh

	#JAVA可执行路径
	JAVA=/web/jdk1.7.0_67/bin/java
	CLASSPATH=/web/jdk1.7.0_67/lib:/web/jdk1.7.0_67/jre/lib

	#JVM设置
	JAVA_OPTIONS="$JAVA_OPTS -server -Xms512m -Xmx512m -XX:NewSize=256m -XX:MaxNewSize=256m -XX:PermSize=128m -XX:MaxPermSize=256m -XX:SurvivorRatio=16 -XX:MaxTenuringThreshold=5 -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSMaxAbortablePrecleanTime=500 -XX:+CMSClassUnloadingEnabled -verbose.gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:$CATALINA_HOME/logs/gc.log -Djava.awt.headless=true"

	#校验方式
	$JETTY_HOME/bin/jetty.sh check

2.添加虚拟目录

	#在etc/jetty.conf中添加新的XML配置选项
	eim-server.xml

	#cat eim-server.xml
	<?xml version="1.0"?>
	<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure_9_0.dtd">
	
	<Configure id="Contexts" class="org.eclipse.jetty.server.handler.ContextHandlerCollection">
	  <Call name="addHandler">
	    <Arg>
	      <New class="org.eclipse.jetty.webapp.WebAppContext">
		<Set name="contextPath">/eim-server</Set>
		<Set name="resourceBase">/web/eln4share/eim-server</Set>
	      </New>
	    </Arg>
	  </Call>
	</Configure>

3.应用下若是启用log4j.properties，注意日志路径
	
	#这是相对路径
	log4j.appender.dailyRoll.file=log/eim-server.log	
	
4.启动服务

	bin/jetty.sh start


	



	
