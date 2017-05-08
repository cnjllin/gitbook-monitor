# Grafana

[官方文档地址](http://docs.grafana.org)

## 安装
Grafana的默认端口是3000，默认账号和密码都是admin。

### CentOS7
在CentOS7上部署Grafana可以使用我写的[ansible role](https://galaxy.ansible.com/frank6866/grafana/),不过为了熟悉Grafana,建议还是手工安装一遍。

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

## macOS
```
brew install grafana
brew services start grafana
```



# 基本概念
参考: [http://docs.grafana.org/guides/basic_concepts/](http://docs.grafana.org/guides/basic_concepts/)

## Data Source
数据源，Grafana支持支持很多种时间序列的数据源，每一种数据源都有一个特定的定制的查询编辑器。  

官方支持的数据源包括： Graphite、InfluxDB、OpenTSDB、Prometheus、Elasticsearch、CloudWatch。

## Organization
Grafana支持多种组织，因此它可以支持多种部署模型，包括只使用一个Grafana为多个潜在的不被信任的组织提供服务。  

在很多场景下，Grafana只有一个组织，每个组织都有一个或者多个数据源。  

所有的Dashboard只被一个特定的组织拥有。

## User
用户是Grafana中的一个有名称的账号，一个用户可以属于一个或者多个组织，在不同的组织中可以有不同的权限。  

Grafana支持多种内部和外部认证的方式，比如从Grafana自己的数据库中，或者从一个外部的SQL数据库，或者从一个外部的LDAP服务器。

## Row
行在Dashboard内部是一个逻辑分割条，用来把Panels组合在一起。

## Panel
面板是Grafana中可视化的基本单元，每个面板都配有一个查询编辑器。面板的类型如下：    

* Graph：可以根据metrics制作图表
* Signlestate
* Table
* Dashlist：没有连接到任何数据源
* Text：没有连接到任何数据源


## Query Editor
查询编辑器向外暴露了数据源，使用查询编辑器我们可以查询数据源包含的metrics。  

我们可以在查询编辑器中使用模板变量。

## Dashboard
控制台把所有的东西聚合在一起，我们可以把控制台当成面板或者行的集合。












