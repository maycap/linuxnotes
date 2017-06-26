## Zstack VM change IP


#### zstack修改虚拟机IP(zstck-cli)，同个L3network修改方案

>登录授权

	admin >>>LogInByAccount  accountName=admin password=xxxxx
	{
	    "inventory": {
	        "accountUuid": "xxx",
	        "createDate": "May 17, 2017 6:04:15 PM",
	        "expiredDate": "May 17, 2017 8:04:15 PM",
	        "userUuid": "xxx",
	        "uuid": "xxx"
	    },
	    "success": true
	}

>查询对应IP的虚拟网卡信息（主要为uuid，vmInstanceUuid）
	
	admin >>>QueryVmNic ip=10.57.17.187
	{
	    "inventories": [
	        {
	            "createDate": "May 17, 2017 2:39:53 PM",
	            "deviceId": 0,
	            "gateway": "10.57.17.1",
	            "ip": "10.57.17.187",
	            "l3NetworkUuid": "647a095c597c4a5e86724a7f2f07667b",
	            "lastOpDate": "May 17, 2017 2:39:53 PM",
	            "mac": "fa:a0:72:4f:32:00",
	            "netmask": "255.255.255.0",
	            "uuid": "e84c6a49f39c470ca506c274efcc92d4",
	            "vmInstanceUuid": "0949299644824113894b2de3983cdb65"
	        }
	    ],
	    "success": true
	}

>获取相关tag（使用vmInstanceUuid）

	admin >>>QuerySystemTag resourceUuid=0949299644824113894b2de3983cdb65
	{
	    "inventories": [
	        {
	            "createDate": "May 17, 2017 2:38:10 PM",
	            "inherent": false,
	            "lastOpDate": "May 17, 2017 2:38:10 PM",
	            "resourceType": "VmInstanceVO",
	            "resourceUuid": "0949299644824113894b2de3983cdb65",
	            "tag": "staticIp::647a095c597c4a5e86724a7f2f07667b::10.57.17.187",
	            "type": "System",
	            "uuid": "36896b36553341a9a4d674bb9f6355ff"
	        }
	    ],
	    "success": true
	}

>卸载虚拟机三层网络，可页面操作 (使用QueryVmNic的uuid)

	admin >>>DetachL3NetworkFromVm vmNicUuid=e84c6a49f39c470ca506c274efcc92d4

>创建IP标签（复制QuerySystemTag，修改下IP即可）

	admin >>>CreateSystemTag resourceUuid=0949299644824113894b2de3983cdb65 resourceType=VmInstanceVO   tag=staticIp::647a095c597c4a5e86724a7f2f07667b::10.57.17.188
	{
	    "inventory": {
	        "createDate": "May 17, 2017 6:37:53 PM",
	        "inherent": false,
	        "lastOpDate": "May 17, 2017 6:37:53 PM",
	        "resourceType": "VmInstanceVO",
	        "resourceUuid": "0949299644824113894b2de3983cdb65",
	        "tag": "staticIp::647a095c597c4a5e86724a7f2f07667b::10.57.17.188",
	        "type": "System",
	        "uuid": "d10558408541421cb21b4d192779af33"
	    },
	    "success": true
	}

>挂载虚拟机三层网络,可页面操作 （l3NetworkUuid为上述的tag中的uuid)

	admin >>>AttachL3NetworkToVm l3NetworkUuid=647a095c597c4a5e86724a7f2f07667b  vmInstanceUuid=0949299644824113894b2de3983cdb65
	

>不用重启，也可生效，主机修改由安装zabbix统一修改与注册