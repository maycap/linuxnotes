#psql_use

***
记录关于数据库的一些操作方法，便于必要时刻使用



>更新字段中某个字符，比如ip地址修改

	#将report_name字段的日期字符替换为'XXXX'

	UPDATE t_xfs_report_condition
	SET report_name = REPLACE (
		report_name,
		'2015/11/9',
		'XXXX'
	)
	WHERE
		report_id = 'c8e5e0ec8ef540e7a37d7bd18368d082'