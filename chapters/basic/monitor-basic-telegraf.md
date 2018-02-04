# Telegraf

监控系统一般如下几个模块：

* 数据采集
* 数据存储
* 数据展示
* 告警

开源的监控方案:
采集数据(Telegraf) -> 存储数据(InfluxDB) -> 显示数据(Grafana)

Telegraf是一个用Golang编写的代理程序，可以收集系统和服务的统计数据，并写入到InfluxDB。Telegraf具有内存占用小的特点，通过开发插件我们可以添加我们需要的服务。

2017.3使用的telegraf版本为V1.2,[官网文档地址](https://docs.influxdata.com/telegraf/v1.2/)。



## 安装
### CentOS7
在CentOS7上部署Telegraf可以使用[ansible role](https://galaxy.ansible.com/frank6866/telegraf/),不过为了熟悉Telegraf,建议还是手工安装一遍。


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

### 测试
当Telegraf启动时，如果InfluxDB中没有Telegraf数据库，会自动创建一个名为telegraf的库。

```
telegraf -config /usr/local/etc/telegraf.conf -test

telegraf -sample-config -input-filter cpu:mem:disk -output-filter influxdb > telegraf.conf
```


## 数据采集频率
默认的采集频率是10s,有点长,可以修改为5s

```
[agent]
  ## Default data collection interval for all inputs
  interval = "10s"
```

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
telegraf -sample-config -input-filter cpu:mem:disk -output-filter influxdb > telegraf.conf

查看telegraf抓取的所有数据

```
# telegraf -test
* Plugin: inputs.system, Collection 1
> system,host=nl-cloud-dyc-k8s-1 load1=0.19,load5=0.34,load15=0.25,n_users=2i,n_cpus=2i 1494408513000000000
> system,host=nl-cloud-dyc-k8s-1 uptime=1902973i,uptime_format="22 days,  0:36" 1494408513000000000
* Plugin: inputs.net, Collection 1
......
```

查看telegraf抓取的内存和网络数据(-input-filter的值为配置文件中inputs.xxx中的xxx,可以设置多个值,使用冒号分隔):

```
# telegraf -test -input-filter mem:diskio
* Plugin: inputs.mem, Collection 1
> mem,host=nl-cloud-dyc-k8s-1 available=1193779200i,used=2950987776i,active=3256971264i,total=4144766976i,free=143593472i,cached=1392947200i,buffered=0i,inactive=581951488i,used_percent=71.19791759313613,available_percent=28.802082406863878 1494408643000000000
* Plugin: inputs.diskio, Collection 1
> diskio,name=vda,host=nl-cloud-dyc-k8s-1 writes=11154527i,read_bytes=2805115392i,write_bytes=40910338048i,read_time=512738i,write_time=49621289i,io_time=35475332i,iops_in_progress=0i,reads=43120i 1494408643000000000
> diskio,name=vda1,host=nl-cloud-dyc-k8s-1 read_time=512595i,write_time=49525882i,io_time=35426178i,iops_in_progress=0i,reads=42936i,writes=10720821i,read_bytes=2804361728i,write_bytes=40910338048i 1494408643000000000
> diskio,name=dm-0,host=nl-cloud-dyc-k8s-1 io_time=101894i,iops_in_progress=0i,reads=176074i,writes=123313i,read_bytes=3152873472i,write_bytes=4582692352i,read_time=135021i,write_time=1289091i 1494408643000000000
```



# Input
telegraf是input驱动的，它从配置文件中指定的input中收集metrics。

使用某些input生成配置文件:

```
# telegraf -sample-config -input-filter cpu:mem:disk -output-filter influxdb > telegraf.conf
```

