##rpmbuild

###preface
使用rpmbuild打包一些软件，定制一些配置与脚本，便于线上安装。

###使用步骤
>获取软件包

	yum install rpmdevtools

>初始化目录树

	rpmdev-setuptree
	
>新建编译配置文件

	cd ~/rpmbuild/SPECS
	rpmdev-newspec squid

>squid.spec参考

	Name:           squid
	Version:		3.5.21
	Release:        1
	Summary:        squid http proxy
	
	Group:          System Environment/Daemons
	License:        GPLv2+ and (LGPLv2+ and MIT and BSD and Public Domain)
	URL:            http://www.squid-cache.org/
	Source0:        %{name}-%{version}.tar.gz
	BuildRoot:	%{_tmppath}/%{name}-%{version}-%{release}-root-%(%{__id_u} -n)
	
	%define squid_dir /home/admin/squid
	%define squid_conf /home/admin/squid.conf
	%define squid_deny_dst /home/admin/deny_dst
	%define squid_deny_dstdomain /home/admin/deny_dstdomain
	%define squid_init	/home/admin/squid.init
	
	#BuildRequires: 	openldap-devel
	#Requires:      	bash >=2.0
	
	%description
	
	
	%prep
	%setup -q
	
	
	%build
	%configure --prefix=%{squid_dir}  --exec-prefix=%{squid_dir} --bindir=%{squid_dir}/usr/bin --sbindir=%{squid_dir}/usr/sbin --sysconfdir=%{squid_dir}/etc --datadir=%{squid_dir}/usr/share --libdir=%{squid_dir}/usr/lib64 --libexecdir=%{squid_dir}/usr/libexec --localstatedir=%{squid_dir}/var --sharedstatedir=%{squid_dir}/var/lib --mandir=%{squid_dir}/usr/share/man --infodir=%{squid_dir}/usr/share/info  --enable-gnuregex  --enable-icmp  --enable-linux-netfilter  --enable-default-err-language="Simplify_Chinese"  --enable-kill-parent-hack  --enable-cache-digests  --enable-dlmalloc  --enable-poll  --enable-async-io=240  --with-large-files  --enable-auth  --enable-auth-basic=DB,LDAP,NCSA,NIS,PAM,POP3,RADIUS,SASL,SMB,getpwnam  --enable-auth-digest=file,LDAP  --enable-auth-negotiate=kerberos,wrapper --enable-digest-auth-helpers=password,ldap
	make -j 2
	
	
	%install
	rm -rf $RPM_BUILD_ROOT
	make install DESTDIR=$RPM_BUILD_ROOT
	cp %{squid_conf} $RPM_BUILD_ROOT%{squid_dir}/etc
	cp %{squid_deny_dst} $RPM_BUILD_ROOT%{squid_dir}/etc
	cp %{squid_deny_dstdomain} $RPM_BUILD_ROOT%{squid_dir}/etc
	cp %{squid_init} $RPM_BUILD_ROOT%{squid_dir}/usr/sbin
	
	
	%clean
	rm -rf $RPM_BUILD_ROOT
	
	
	%files
	%defattr(-,admin,admin,-)
	%{squid_dir}/etc/*
	%{squid_dir}/usr/*
	%{squid_dir}/var/*
	%doc
	
	
	%post
	mkdir -p /home/admin/squid/var/spool/squid
	chown -R admin.admin /home/admin/squid
	mv /home/admin/squid/usr/sbin/squid.init /etc/init.d/squid
	
	
	%changelog
	* Mon Sep 12 2016  <gencat@163.com> 3.5.21-1
	- Initial RPM release

>spec一些配置记录

	%define 用于自定义变量，便于统一修改
	
	%configure 单指定 prefix，rpm安装并不在指定目录，需要多项指定目录
	
	%install中可以后跟命令，修改一些配置与文件内容
	
	%file需要自己定义哪些文件要加入，否则无法打包成功，也不会报错
	
	%post是rpm安装完毕后执行的操作，可用于修改属性，移动脚本之类的
	
	