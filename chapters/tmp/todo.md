



## mysql
servers = ["root@tcp(127.0.0.1:3306)/?tls=false"]


Mar 13 13:40:28 nl-cloud-openstack-1 telegraf: 2017/03/13 13:40:28 E! Error parsing /etc/telegraf/telegraf.conf, line 1213: field corresponding to `gather_innodb_metrics' is not defined in `*mysql.Mysql'





# TODO

各种监控指标的配置脚本


如何使用主机名过滤



计算网卡流量、pps


SELECT mean("commands_commit")+mean("commands_rollback") FROM "mysql" WHERE $timeFilter GROUP BY time($interval) fill(null)


mean("commands_commit")
mean("commands_rollback")


(mean("commands_commit")+mean("commands_rollback"))




# 告警
只有Grafana v4.0及其以上版本才支持告警。

打开一个panel,切换到Alter标签。


# todo
## ceph osd used table
SELECT LAST("used_kb") / LAST("total_kb") AS "OSD" FROM "script.ceph.osd" WHERE "cluster_name" =~ /^$cluster_name$/ AND $timeFilter GROUP BY time($interval), "osd" fill(null)

options中选择time seires aggregation







