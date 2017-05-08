# Telegraf


Telegraf是一个用Golang编写的代理程序，可以收集系统和服务的统计数据，并写入到InfluxDB。Telegraf具有内存占用小的特点，通过开发插件我们可以添加我们需要的服务。

本文使用的telegraf版本为V1.2,[官网文档地址](https://docs.influxdata.com/telegraf/v1.2/)。



## 安装
### CentOS7
在CentOS7上部署Telegraf可以使用我写的[ansible role](https://galaxy.ansible.com/frank6866/telegraf/),不过为了熟悉Telegraf,建议还是手工安装一遍。


创建仓库文件  

```
cat <<EOF | sudo tee /etc/yum.repos.d/influxdata.repo
[influxdata]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```

install, start and enable service: 

```
yum install telegraf
systemctl start telegraf
systemctl enable telegraf
```

配置文件路径:  **/etc/telegraf/telegraf.conf**

### macOS
```
brew install telegraf
brew services start telegraf
```

配置文件路径:  **/usr/local/etc/telegraf.conf**


## 和InfluxDB集成
以CentOS7为例。

和InfluxDB集成的配置主要在/etc/telegraf/telegraf.conf文件的**[[outputs.influxdb]]**块中

### urls配置
配置InfluxDB的API的endpoint，比如:  

```
  urls = ["https://192.168.168.201:8086"]
```

### auth配置
如果InfluxDB启用了auth，在telegraf中需要做如下配置:  

```
  username = "frank6866"
  password = "change_me"
```  

注意，如果InfluxDB中没有启用认证，可以不用事先在InfluxDB中创建telegraf使用的数据库，telegraf启动后会去创建；如果InfluxDB中启用了认证，并且telegraf配置的用户在InfluxDB中没有创建数据库的权限，telegraf会报无法创建数据库的错误，如下:  

```
# journalctl -fl -u telegraf
......
InfluxDB Output Error: {"error":"database not found: \"telegraf\""}
......
```

解决: 在InfluxDB中使用具有admin权限的账户创建数据库。

```
> CREATE DATABASE telegraf
```

### https配置
如果InfluxDB启用了https，在telegraf中需要做如下配置:  

```
  ssl_ca = "/etc/pki/influxdb/cacert.crt"
  ssl_cert = "/etc/pki/influxdb/influxdb.crt"
  ssl_key = "/etc/pki/influxdb/private/influxdb.key"
```




## 验证
telegraf -config /usr/local/etc/telegraf.conf -test

telegraf -sample-config -input-filter cpu:mem:disk -output-filter influxdb > telegraf.conf



# Input
telegraf是input驱动的，它从配置文件中指定的input中收集metrics。



telegraf -sample-config -input-filter cpu:mem:disk:net:mysql -output-filter influxdb > telegraf.conf.template


servers = ["root@tcp(127.0.0.1:3306)/?tls=false"]


## mysql
Mar 13 13:40:28 nl-cloud-openstack-1 telegraf: 2017/03/13 13:40:28 E! Error parsing /etc/telegraf/telegraf.conf, line 1213: field corresponding to `gather_innodb_metrics' is not defined in `*mysql.Mysql'














