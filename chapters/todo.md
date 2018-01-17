# 硬件监控

## io
网卡光衰
bond模式是否有双网卡
ssd寿命监控
raid卡状态监控
raid卡电池监控



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



* Kubernetes监控
* Ceph监控
* OpenStack监控





## tcp connection

已连接
time-wait
fina-wait


带宽利用率监控


* [Telegraf](chapters/docker/monitor-docker-telegraf.md)





# 分类
## web server
apache

haproxy

httpd_listener

http_response

httpjson

nginx

net_response/

## db、mq及缓存等
### influxdb

### mysql/mariadb




### mongo

### redis

### rabbitmq


### memcached


### kafka


### etcd

### zk


### cassandra


## elk
### Elasticsearch

###


























