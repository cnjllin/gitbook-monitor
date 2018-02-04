# Grafana

Grafana是一个开源的监控数据展示和分析工具,界面美观有科技感,功能强大,可以从多种数据源取值并进行展示,比如[InfluxDB](https://frank6866.gitbooks.io/monitor/content/chapters/basic/monitor-basic-influxdb.html).

[官方文档地址](http://docs.grafana.org)

## 安装
Grafana的默认端口是3000，默认账号和密码都是admin。

### CentOS7
在CentOS7上部署Grafana可以使用[ansible role](https://galaxy.ansible.com/frank6866/grafana/),不过为了熟悉Grafana,建议还是手工安装一遍。

yum仓库文件

```
[grafana]
name=grafana
baseurl=https://packagecloud.io/grafana/stable/el/7/$basearch
repo_gpgcheck=0
enabled=1
gpgcheck=0
gpgkey=https://packagecloud.io/gpg.key https://grafanarel.s3.amazonaws.com/RPM-GPG-KEY-grafana
sslverify=0
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

安装grafana:  

```
# yum install grafana
```


启动Grafana并设置为开机启动:  

```
# systemctl start grafana-server
# systemctl enable grafana-server
```

* CentOS7中Grafana配置文件位置是:  **/etc/grafana/grafana.ini**
* CentOS7中Grafana日志文件位置是:  **/var/log/grafana/grafana.log**

### macOS
```
brew install grafana
brew services start grafana
```

配置文件位置:

```
grafana-server --config=/usr/local/etc/grafana/grafana.ini --homepath /usr/local/share/grafana cfg:default.paths.logs=/usr/local/var/log/grafana cfg:default.paths.data=/usr/local/var/lib/grafana cfg:default.paths.plugins=/usr/local/var/lib/grafana/plugins
```


### 将grafana放到nginx后面

```
nginx.conf

location /grafana/ {
     proxy_pass         http://localhost:3000/;
     proxy_set_header   Host $host;
}

grafana.ini update root:

[server]
root_url = %(protocol)s://%(domain)s:%(http_port)s/grafana/
```




## 基本概念
参考: [http://docs.grafana.org/guides/basic_concepts/](http://docs.grafana.org/guides/basic_concepts/)

### Data Source
数据源，Grafana支持支持很多种时间序列的数据源，每一种数据源都有一个特定的定制的查询编辑器。  

官方支持的数据源包括： Graphite、InfluxDB、OpenTSDB、Prometheus、Elasticsearch、CloudWatch。

### Organization
Grafana支持多种组织，因此它可以支持多种部署模型，包括只使用一个Grafana为多个潜在的不被信任的组织提供服务。  

在很多场景下，Grafana只有一个组织，每个组织都有一个或者多个数据源。  

所有的Dashboard只被一个特定的组织拥有。

### User
用户是Grafana中的一个有名称的账号，一个用户可以属于一个或者多个组织，在不同的组织中可以有不同的权限。  

Grafana支持多种内部和外部认证的方式，比如从Grafana自己的数据库中，或者从一个外部的SQL数据库，或者从一个外部的LDAP服务器。

### Row
行在Dashboard内部是一个逻辑分割条，用来把Panels组合在一起。

### Panel
面板是Grafana中可视化的基本单元，每个面板都配有一个查询编辑器。面板的类型如下：    

* Graph：可以根据metrics制作图表
* Signlestate
* Table
* Dashlist：没有连接到任何数据源
* Text：没有连接到任何数据源


### Query Editor
查询编辑器向外暴露了数据源，使用查询编辑器我们可以查询数据源包含的metrics。  

我们可以在查询编辑器中使用模板变量。

### Dashboard
控制台把所有的东西聚合在一起，我们可以把控制台当成面板或者行的集合。

### 模板(Template)
定义模板变量:

这里以InfluxDB(使用Telegraf采集)数据源为例，配置主机名变量
点击Dashboard上方的小齿轮('Manage dashboard' -> 'Templating' -> 'New')

* Variable -> Name:变量的名称(在查询条件中显示,这里设置为server_name)
* Variable -> Label:变量的标签名(在页面上过滤时显示,这里设置为主机名)
* "Query Options" -> "Data Source"：选择我们配置的数据源
* "Query Options" -> "Refresh"：选择"On Time Range Change"
* "Query Options" -> "Query"： 查询主机名，输入 SHOW TAG VALUES ON "telegraf" WITH KEY = "host"
* "Selection Options" -> "Multi-value": 勾选中

使用模板变量, 以CPU为例:

```
FROM default cpu WHERE host =~ /^$server_name$/
SELECT field(usage_user) mean()
GROUP BY time($interval) tag(host)
cpu.usage($tag_host)

```



## 配置
http://docs.grafana.org/installation/configuration/

## SMTP配置
[http://docs.grafana.org/installation/configuration/#smtp](http://docs.grafana.org/installation/configuration/#smtp)

smtp部分配置如下:

```
[smtp]
enabled = true
host = smtp.vip.sina.com:25
user = dyc_abc
password = ...
;cert_file =
;key_file =
;skip_verify = false
from_address = dyc_abc@vip.sina.com
```

修改完配置文件后需要重启grafana

> systemctl restart grafana

### 异常
测试发送邮件时,发现发送失败,查看/var/log/grafana/grafana.log文件存在以下问题

> Notification service failed to initialize" logger=server erro="Invalid email address for smpt from_adress config

解决: 我的from_address配置的是**admin@grafana.10.12.10.8**,不是一个正常的email地址,修改为admin@grafana_10_12_10_8.com


> error="gomail: could not send email 1: 553 Envolope sender mismatch with login user.."

解决: user字段必须和from_address字段中@前面的一样


## snippets

```
SELECT mean("commands_commit")+mean("commands_rollback") FROM "mysql" WHERE $timeFilter GROUP BY time($interval) fill(null)

mean("commands_commit")
mean("commands_rollback")
(mean("commands_commit")+mean("commands_rollback"))

```


## todo
### ceph osd used table

```
SELECT LAST("used_kb") / LAST("total_kb") AS "OSD" FROM "script.ceph.osd" WHERE "cluster_name" =~ /^$cluster_name$/ AND $timeFilter GROUP BY time($interval), "osd" fill(null)

options中选择time seires aggregation
```

