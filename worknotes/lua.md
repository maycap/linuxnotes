### LUA

#### 前言
nginx_lua 简单笔记


#### 安装

>nginx编译参考
	
	--prefix=/web/nginx --with-pcre --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_stub_status_module --with-file-aio --with-ipv6 --with-poll_module --with-select_module --add-module=/web/tools/echo-nginx-module-0.59 --add-module=/web/tools/lua-nginx-module-0.10.5 --add-module=/web/tools/ngx_devel_kit-0.3.0 --add-module=/web/tools/redis2-nginx-module-0.13 --add-module=/web/tools/set-misc-nginx-module/

>openresty

#### 实例

>读文件获取参数

	location /getvkup {
		default_type 'text/html';
		content_by_lua_file  conf/getvkup.lua;
	}		

	
getvkup.lua内容参考

	local myvkup={}
	local json = require("json")
	local file="/web/nginx/conf/vkupstream.conf"
	
	local uptype="vk_%a"
	
	function get_vkup(file)

		local lines={}
		local f=io.open(file,'r')
		for line in f:lines() do
	
			local tag=string.find(line,uptype)
			if(tag)
			then
				table.insert(myvkup,string.sub(line, string.find(line,uptype)))
			end
		end
		f:close()
	end

	get_vkup(file)
	local result=json.encode(myvkup)
	ngx.say(result)


		

