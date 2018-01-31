---
description: InfluxDB是一个开源的、分布式时序、事件和指标数据库。InfluxDB使用Go语言编写，着力于高性能地查询与存储时序型数据，无需外部依赖。
---
# InfluxDB
InfluxDB是一个开源的、分布式时序、事件和指标数据库。InfluxDB使用Go语言编写，着力于高性能地查询与存储时序型数据，无需外部依赖。

本文使用InfluxDB版本为V1.2，[官网文档地址](https://docs.influxdata.com/influxdb/v1.2/)。

InfluxDB有以下特点:

* schemaless(无结构): 可以是任意数量的列
* scalable: 可扩展的
* 有很多聚合函数: min, max, sum, count, mean等


## 概念
| 名称 | 解释 
|:----|------
| user | 在InfluxDB中有两种类型的user：Admin用户对所有的数据库都有读和写的权限，并且可以执行管理员才能执行的查询和用户管理命令；非admin用户可以对某个或者某几个数据库，具有读或者写或者读写的权限。
| schema | 在InfluxDB中数据是如何存储和组织的。InfluxDB的基础schema包括databases、retention policies、series、measurements、tag keys、tag values和field keys。
| field | InfluxDB数据结构中的key-value pair(键值对)，用来记录元数据(metadata)和实际的数据值。InfluxDB中的数据结构需要field，并且field不能被索引。针对field的值的查询将会扫描满足特定时间需求的所有点。所以，和tag相比，field没有tag的查询效率高。
| field key | field key是key-value pair中的key部分，filed key是字符串，field key存储的是元数据。
| field value | field value是key-value pair中的value部分，field value是实际的数据，可以是字符串、浮点型、整型或者布尔型。一个field value总是和一个timestamp关联。field value不会被索引，对field value的查询将会扫描满足特定时间需求的所有点，是很不高效的。(提示：在查询中使用tag value而不是field value)
| field set | 一个point上很多key-value pair(field)的集合。
| point | 点，series中很多field的集合，相当于关系数据库中的一行数据。point是由timestamp、fields和tags组成。每一个point通过它的series和timestamp唯一标识。在一个series中，对于一个timestamp我们不能存储多个point。
| function | InfluxQL中的aggregations、selectors和transformations。
| continuous query(CQ，持续查询) | continuous query在数据库中自动和周期地运行。continuous query在SELECT中需要一个aggregation(聚合函数)，并且必须包含**GROUP BY time()**。
| database(数据库) | 数据库是一个逻辑的容器，包括：users、retention policies、continuous queries和time series data。
| measurement | 相当于关系数据库中的table,measurement是tag、field和time的容器。对于measurement来说,field是必须的,并且不能根据field来排序。
| retention policy | 保留策略,用于决定要保留多久的数据,保存几个备份,以及集群的策略。对于每一个database来说，只有一个retention policy。当我们创建一个database时，InfluxDB会自动创建一个叫**autogen**的retention policy，其duration为无穷，replication factor是1，shard group duration是7天。
| duration(区间) | duration是retention policy的一个属性，用来决定InfluxDB将数据存储多久，比duration更老的数据将会被自动删掉。
| replication factor(复制系数) |  replication factor是retention policy的一个属性，用来决定InfluxDB集群将数据存储多少个副本。（Replication factor不支持单节点的集群）
| series | 一个series可以理解为一个曲线图，这个曲线图上的所有点的如下属性必须是一样的：measurement、tag set和retention policy，如果有一个不一样，就是另外一个series。举个例子，在一个measurement中，retention policy都是一样的，tag(k)的取值共有f(k)，那么在这个measurement中，共有f(1) * f(2) * ... * f(k)个series。注意，field set不是series的标识符，不能用来区别series。
| tag | tag是InfluxDB中记录元数据的的key-value pair。tag是可选的,tag会被索引,所以通常tag被用来存储经常被查询的的元数据。tag是以字符串的形式存放的。
| tag key | tag的key，使用字符串标识；tag的key被索引了，所以查询起来很高效。
| tag value | tag的value，使用字符串标识；tag的value被索引了，所以查询起来很高效。
| tag set | 一个point上tag key和tag value的结合
| timestamp | 和一个point关联的日期和时间，在InfluxDB中，所有的时间都是UTC时间。
| tsm(Time Structured Merge tree) | tsm是InfluxDB的数据存储格式。和B+树或者LSM树相比，TSM允许更高的数据压缩率和更高的读写吞吐。
| aggregation | 聚合函数，聚合函数对一组point做聚合并返回一个聚合值(比如对一组point求平均值的函数mean)。
| transformation | transformation是InfluxQL的一类函数，它可以计算一些特定的point，返回一个或者一组值；但是transformation函数不返回aggregation值。
| batch | 批处理，一组point通过换行符分隔，这组point可以通过HTTP请求被一起提交到数据库中。这样可以让HTTP API更加高效。InfluxDB推荐batch size在5000-10000之间；针对不同的使用场景也可以调整。
| identifier | 数据库、field key、measurement name、retention policy name、tag keys等的唯一标识符。
| line protocol | 将points写入到InfluxDB时，基于文本的协议。
| server | 一个运行InfluxDB的机器(物理机或者虚拟机)。对于每一个server来说，只能有一个InfluxDB进程。
| node | 一个独立的**influxd**进程。
| metastore | 包含InfluxDB运行状态的系统信息。metastore包含user、databases、retention policies、shard metadata、continuous queries和subscriptions
| now() | 服务器的时间戳，精确到nanosecond(纳秒)
| values per second | 一个InfluxDB服务的监控指标，表示数据被持久化到InfluxDB中的速率。values per second的计算方法为：每秒写入的point个数乘以每个point的field的个数。比如，某个point有4个field，每秒可以执行10次批处理操作，每次批处理操作的大小为5000；那么values per second = 4 * 10 * 5000 = 200,000
| wal(Write Ahead Log) | 对最近写入的数据的临时缓存。为了减少存储文件被访问的频率，InfluxDB将新的point缓存到WAL中，直到WAL的的缓存空间满了或者缓存的point老化而触发写到磁盘文件中去的操作。WAL中的point可以被查询，在系统重启后也存在；但重启后，InfluxDB进程启动了，WAL中的所有point必须被flush到磁盘上，InfluxDB才能接收新的写请求。


## 安装
### CentOS7
在CentOS7上部署InfluxDB可以使用我写的[ansible role](https://galaxy.ansible.com/frank6866/influxdb/),不过为了熟悉InfluxDB,建议还是手工安装一遍。

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
# yum install influxdb
# systemctl start influxdb
# systemctl enable influxdb
```

> CentOS7上InfluxDB的配置文件路径是:  **/etc/influxdb/influxdb.conf**


如果启动失败

```
# systemctl status influxdb
● influxdb.service - InfluxDB is an open-source, distributed, time series database
   Loaded: loaded (/usr/lib/systemd/system/influxdb.service; enabled; vendor preset: disabled)
   Active: failed (Result: start-limit)
     Docs: https://docs.influxdata.com/influxdb/
  Process: 22232 ExecStart=/usr/bin/influxd -config /etc/influxdb/influxdb.conf $INFLUXD_OPTS (code=exited, status=1/FAILURE)
 Main PID: 22232 (code=exited, status=1/FAILURE)
```

解决思路: 现在终端中用命令启动一下influxdb, 命令systemd中的ExecStart的值:

```
/usr/bin/influxd -config /etc/influxdb/influxdb.conf $INFLUXD_OPTS
```

很奇怪的是, 用命令可以启动influxdb, 但是无法用systemd启动influxdb, 查看/var/log/messages

```
open server: open tsdb store: open /influxdb/data/_internal: permission denied
```

发现/influxdb/data的owner是root, 所以在终端中可以使用命令行启动; 为了能在systemd中启动



### macOS
```
➜ brew install influxdb
➜ brew services start influxdb
```

> macOS上InfluxDB的配置文件路径是:  **/usr/local/etc/influxdb.conf**


# 配置
## 常用端口
* TCP 8083: web ui使用的端口
* TCP 8086: HTTP API使用的端口
* TCP 8088: remote backups and restores

## admin ui
在1.3版本中将会移除admin ui功能，不要再依赖这个功能。  

## HTTPS配置
### 生成证书
这里用的是在macOS上自建的CA，用来签发证书。(如何搭建CA，参考我的[搭建CA](https://frank6866.gitbooks.io/linux/content/chapters/security/security-ca-setup.html)这篇文章)

```
➜ mkdir /tmp/process_csr/influxdb/
➜ openssl genrsa -out /tmp/process_csr/influxdb/influxdb.key 2048
➜ openssl req -new -batch -days 3650 -subj /CN=influxdb -key /tmp/process_csr/influxdb/influxdb.key -reqexts SAN -config <(cat /usr/local/etc/openssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=IP:192.168.168.201,IP:192.168.168.202,IP:192.168.168.203,DNS:localhost")) -out /tmp/process_csr/influxdb/influxdb.csr
➜ openssl ca -in /tmp/process_csr/influxdb/influxdb.csr  -extensions SAN -config <(cat /usr/local/etc/openssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=IP:192.168.168.201,IP:192.168.168.202,IP:192.168.168.203, DNS:localhost"))  -out /tmp/process_csr/influxdb/influxdb.crt
```

将生成的证书及key文件拷贝到InfluxDB服务器上。

### 修改配置

编辑/etc/influxdb/influxdb.conf文件的[http]部分，修改三个配置项:  

* https-enabled: 表示是否启用https(默认为false)
* https-certificate: 证书文件路径
* https-private-key: 证书对应的密钥文件路径 

修改完后，如下:  

```
[http]
...
  # Determines whether HTTPS is enabled.
    https-enabled = true

  # The SSL certificate to use when HTTPS is enabled.
    https-certificate = "/etc/pki/influxdb/influxdb.crt"

  # Use a separate private key location.
    https-private-key = "/etc/pki/influxdb/private/influxdb.key"
...
```

修改完后重启InfluxDB:  

```
# systemctl restart influxdb
```

### 测试HTTPS
```
# influx -ssl
Connected to https://localhost:8086 version 1.2.2
InfluxDB shell version: 1.2.2
>
```

-ssl选项表示Use https for requests.

可能出现的问题及解决方法:  

* 如果出现"x509: certificate is valid for xxx, not localhost"的问题，确认是否在证书的SAN中添加了DNS:localhost
* 如果出现"x509: certificate signed by unknown authority"，确认CA的证书被系统信任((CentOS7或macOS中如何添加对CA证书的信任，参考我的[搭建CA](https://frank6866.gitbooks.io/linux/content/chapters/security/security-ca-setup.html)这篇文章)).


## authentication
参考[官网文档](https://docs.influxdata.com/influxdb/v1.2/query_language/authentication_and_authorization/#authorization)

编辑/etc/influxdb/influxdb.conf文件的[http]部分，修改如下配置项:  

* auth-enabled: 是否启用认证(默认是false)  

修改完后，如下:  

```
......
[http]
......
  # Determines whether HTTP authentication is enabled.
    auth-enabled = true
......
```    

修改完后重启InfluxDB:  

```
# systemctl restart influxdb
```

启用认证后，如果没有admin用户，执行操作时会报错，如下:  

```
# influx -ssl
Connected to https://localhost:8086 version 1.2.2
InfluxDB shell version: 1.2.2
> show databases;
ERR: error authorizing query: create admin user first or disable authentication
Warning: It is possible this error is due to not setting a database.
Please set a database with the command "use <database>".
```

首先创建一个有admin权限用户，比如名为admin的用户(名为admin的用户不是必须的，具有admin权限的用户可以是其他名字):  

```
> CREATE USER admin WITH PASSWORD 'change_me' WITH ALL PRIVILEGES
```


### 在CLI中使用认证
可以在influx命令行中使用auth命令认证(比较麻烦):  

```
> auth
username: admin
password:
> show databases
name: databases
name
----
_internal
```

可以在启动influx时使用认证(通过history可以看到密码，不安全):  

```
# influx -ssl -username admin -password change_me
Connected to https://localhost:8086 version 1.2.2
InfluxDB shell version: 1.2.2
> show databases
name: databases
name
----
_internal

```

通过环境变量的方式(方便，推荐的方式，也便于管理多套influxdb环境，比如开发环境和测试环境可以各写一个rc文件，用的时候source一下rc文件就可以了，用登录到机器上就可以操作；生产环境建议还是老老实实登录到机器上去玩，因为可能会以为操作的是测试环境的库，而实际上操作的是生产的库，避免误操作):  


```
# vi ~/influxdb.rc
export INFLUX_USERNAME=admin
export INFLUX_PASSWORD=change_me
```

可以不用输入账号和密码，如下:  

```
# influx -ssl
Connected to https://localhost:8086 version 1.2.2
InfluxDB shell version: 1.2.2
> show databases;
name: databases
name
----
_internal
```

#### example
在mac上使用influx客户端连接192.168.168.201上的influxdb(配置了ssl):  

```
# vi 192.168.168.201.rc
export INFLUX_HOST=192.168.168.201
export INFLUX_USERNAME=admin
export INFLUX_PASSWORD=change_me
```

测试了一下，想把host也放到rc文件中，这样切换不同环境就更方便了:  

```
➜ influx
Failed to connect to http://localhost:8086: Get http://localhost:8086/ping: dial tcp [::1]:8086: getsockopt: connection refused
Please check your connection settings and ensure 'influxd' is running.
```

但是好像不支持，不过在命令行里面使用-host选项还是可以的，如下:  

```
➜ influx -ssl -host 192.168.168.201
Connected to https://192.168.168.201:8086 version 1.2.2
InfluxDB shell version: v1.2.0
> SHOW GRANTS FOR frank6866
database privilege
-------- ---------
telegraf READ

```

刚开始这里有一个疑问的地方，在192.168.168.201的influxdb上，我配置了HTTPS，在influx的命令行选项里面我没有看到ca相关的选项，但是上面的命令连接成功了。想到在192.168.168.201的influxdb上的证书是用我在macOS上自建的CA签发的，CA的根证书我已经信任过了，所以上面的命令可以连接到192.168.168.201的influxdb上。如果在macOS取消对自建CA根证书的信任，在运行上面的命令，如下:

```
➜ influx -ssl -host 192.168.168.201
Failed to connect to https://192.168.168.201:8086: Get https://192.168.168.201:8086/ping: x509: certificate signed by unknown authority
Please check your connection settings and ensure 'influxd' is running.
```

所以如果碰到"x509: certificate signed by unknown authority"类似的问题，解决思路在系统中添加对证书的信任。(CentOS7或macOS中如何添加对CA证书的信任，参考我的[搭建CA](https://frank6866.gitbooks.io/linux/content/chapters/security/security-ca-setup.html)这篇文章)


