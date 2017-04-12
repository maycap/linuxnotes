##Linux Join AD##

###前言###
一些机器作为开发测试机器，大量人员账户管理很不方便。因此将Linux服务器加入AD托管比较简单，记录一种简单的加入AD域的方式

***
###步骤

>开启ssh登录认证，默认开启

	sshd_config,开启 UsePAM yes

>软件获取地址

	https://github-cloud.s3.amazonaws.com/releases/66377182/03e1c766-fd9d-11e6-8c7d-e0200a083b18.sh?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAISTNZFOVBIJMK3TQ%2F20170316%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20170316T030400Z&X-Amz-Expires=300&X-Amz-Signature=0eabfd1dffc8236a1d82e47a7699a9b59e44e4fa4a24424e1bbe6c33b68a46a8&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dpbis-open-8.5.3.293.linux.x86_64.rpm.sh&response-content-type=application%2Foctet-stream
	
>执行安装

	sh  pbis-open-8.5.3.293.linux.x86_64.rpm.sh
	#一些选择确认（可采用yes，采用默认值）

>修改dns解析,（此DNS是与AD域相关的地址）

	vim /etc/resolv.conf
	nameserver 192.168.0.100
	
	#此dns可以解析下步骤中的AD域（maycap.cn)
	
>加入AD域

	/opt/pbis/bin/domainjoin-cli join maycap.cn administrator

>设置PBIS

	#用户前缀若设置 maycap.cn，则用户全称显示为  MAYCAP.CN//cap.may
	/opt/pbis/bin/config UserDomainPrefix  ""
	#配合上述设置，也设置为空
	/opt/pbis/bin/config DomainSeparator ""
	/opt/pbis/bin/config AssumeDefaultDomain true
	/opt/pbis/bin/config LoginShellTemplate /bin/bash
	/opt/pbis/bin/config HomeDirTemplate %H/%U
	/opt/pbis/bin/config RequireMembershipOf "maycap\\domain^users"
	
>pam修改

	1、pam_unix.so 在 pam_lsass.so 之前
	2、验证采用 sufficient ，通过则返回，不继续使用
	3、session 采用默认配置，session触发认证创建用户家目录
	
***
###pam快速手册###

>常见四种认证
	
<table>
	<tr>
		<td>名称</td>
		<td>别名</td>
		<td>解释</td>	
	</tr>
	<tr>
		<td>session</td>
		<td>会话管理</td>
		<td>主要是提供对会话的管理和记账，AD认证中负责初始化家目录</td>
	</tr>
		<tr>
		<td>password</td>
		<td>密码管理</td>
		<td>主要是用来修改用户的密码</td>		
	</tr>
	<tr>
		<td>auth</td>
		<td>认证管理</td>
		<td>接受用户名和密码，进而对该用户的密码进行认证，并负责设置用户的一些秘密信息
</td>
	</tr>
	<tr>
		<td>account</td>
		<td>帐户管理</td>
		<td>检查帐户是否被允许登录系统，帐号是否已经过期，帐号的登录是否有时间段的限制等等</td>
	</tr>
</table>

>pam验证控制类型

<table border="1">
	<tr>
		<th>Control Flags</th>
		<td>返回值</td>
		<td>是否继续</td>
		<td>影响</td>
	</tr>
	<tr>
		<th rowspan="2">required</th>
		<td>Pass</td>
		<td>是</td>
		<td>Deine by system</td>
	</tr>
	<tr>
		<td>Fail</td>
		<td>是</td>
		<td>Fail</td>
	</tr>
		<tr>
		<th rowspan="2">requisite</th>
		<td>Pass</td>
		<td>是</td>
		<td>Deine by system</td>
	</tr>
	<tr>
		<td>Fail</td>
		<td>否</td>
		<td>Fail</td>
	</tr>
		<tr>
		<th rowspan="2">sufficient</th>
		<td>Pass</td>
		<td>否</td>
		<td>OK</td>
	</tr>
	<tr>
		<td>Fail</td>
		<td>是</td>
		<td>ignore</td>
	</tr>
		<tr>
		<th rowspan="2">optional</th>
		<td>Pass</td>
		<td>是</td>
		<td>ignore</td>
	</tr>
	<tr>
		<td>Fail</td>
		<td>是</td>
		<td>ignore</td>
	</tr>
</table>

>openssh 安装其 sshd PAM 策略，如下所示：

	/etc/pam.d/
	sshd -> system-remote-login -> system-login -> system-auth
	
> 认证模块速查

<table>
	<tr>
		<td>名称</td>
		<td>用途</td>
		<td>备注</td>	
	</tr>
	<tr>
		<td>pam_unix.so</td>
		<td>用于检查用户密码作为认证</td>
		<td>默认情况不允许密码为空的用户进入</td>
	</tr>
	<tr>
		<td>pam_permit.so</td>
		<td>用于检查用户密码作为认证</td>
		<td>允许密码为空的情况</td>		
	</tr>
	<tr>
		<td>pam_lsass.so</td>
		<td>ldap用户认证</td>
		<td></td>
	</tr>

</table>	

***
###备注###
1. AD域与本地账户冲突，因此必须本地认证优先
2. 认证时主机名不能含有域（即 ".xx"，会被识别成xx域）
3. 所有AD账户都在同一个组（可进一步优化）
