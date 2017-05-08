# InfluxQL
使用下面的命令进入InfluxDB交互模式

```
# influx
```

# 元数据管理
## 用户管理
首先创建一个有admin权限用户(加上WITH ALL PRIVILEGES)，比如名为admin的用户(名为admin的用户不是必须的，具有admin权限的用户可以是其他名字):  

```
> CREATE USER admin WITH PASSWORD 'change_me' WITH ALL PRIVILEGES
```

创建一个普通用户

```
> CREATE USER frank6866 WITH PASSWORD 'change_me'
```


查看所有用户

```
> show users
user      admin
----      -----
admin     true
frank6866 false
```

给frank6866用户赋予admin权限(GRANT ALL PRIVILEGES):  

```
> GRANT ALL PRIVILEGES TO frank6866
> show users
user      admin
----      -----
admin     true
frank6866 true
```

撤销frank6888用户的管理员权限(REVOKE ALL PRIVILEGES):  

```
> REVOKE ALL PRIVILEGES from frank6866
> show users
user      admin
----      -----
admin     true
frank6866 false
```

给frank6866在telegraf库中所有权限(所有权限是ALL，读是READ，写是WRITE):  

```
> GRANT ALL ON telegraf TO frank6866
> SHOW GRANTS FOR frank6866
database privilege
-------- ---------
telegraf ALL PRIVILEGES
```

撤销frank6866在telegraf库中的写权限

```
> REVOKE WRITE ON telegraf FROM frank6866
> SHOW GRANTS FOR frank6866
database privilege
-------- ---------
telegraf READ
```

可以看到，如果frank6866用户在某个库中，原本是ALL的权限，被收回WRITE权限后，还剩下READ权限。

修改frank6866用户的密码

```
> SET PASSWORD FOR "frank6866" = 'change@me'
```


## 数据库操作
显示所有数据库

```
> show databases
name: databases
name
----
_internal
telegraf
```

创建一个名为db_test的数据库

```
> create database db_test
```

删除名为db_test的数据库

```
> drop database db_test
```

切换到名为telegraf的数据库

```
> use telegraf
```

## 表操作
### show measurements
显示telegraf库中的所有表

```
> show measurements
name: measurements
name
----
apache
ceph
cpu
......
rabbitmq_node
rabbitmq_overview
rabbitmq_queue
swap
system
```

### show field keys
查看表的字段

查看telegraf库中所有表的所有字段(如果当前use的库是telegraf，可以省略on条件):

```
> show field keys on telegraf
name: rabbitmq_overview
fieldKey           fieldType
--------           ---------
channels           integer
connections        integer
consumers          integer
exchanges          integer
messages           integer
messages_acked     integer
messages_delivered integer
messages_published integer
messages_ready     integer
messages_unacked   integer
queues             integer

name: rabbitmq_queue
fieldKey                  fieldType
--------                  ---------
consumer_utilisation      float
consumers                 integer
idle_since                string
memory                    integer
message_bytes             integer
message_bytes_persist     integer
......
```

查看telegraf库中ceph表的所有字段(如果当前use的库是telegraf，可以省略on条件):

```
> show field keys on telegraf from ceph
name: ceph
fieldKey                                    fieldType
--------                                    ---------
accept_timeout                              float
activating_latency.avgcount                 float
activating_latency.sum                      float
active_latency.avgcount                     float
active_latency.sum                          float
agent_evict                                 float
agent_flush                                 float
agent_skip                                  float
agent_wake                                  float
apply_latency.avgcount                      float
apply_latency.sum                           float
backfilling_latency.avgcount                float
backfilling_latency.sum                     float
begin                                       float
begin_bytes.avgcount                        float
begin_bytes.sum                             float
begin_keys.avgcount                         float
begin_keys.sum                              float
begin_latency.avgcount                      float
begin_latency.sum                           float
buffer_bytes                                float
bytes                                       float
bytes_dirtied                               float
bytes_wb                                    float
...
```


### show tag keys
显示tag keys。命令格式为:

```
show_tag_keys_stmt = "SHOW TAG KEYS" [on_clause] [ from_clause ] [ where_clause ]
                     [ limit_clause ] [ offset_clause ] .
```

显示当前数据库的所有表上的tag key(如果命令操作的是当前库，on条件可以省略；如果要显示db_test数据库上所有的tag的命令是show tag keys on db_test)

```
> show tag keys
name: apache
tagKey
------
host
port
server

name: ceph
tagKey
------
collection
host
id
type

......
```


显示当前数据库上ceph表中的所有tags:

```
> show tag keys from ceph
name: ceph
tagKey
------
collection
host
id
type
```

显示当前数据库上ceph表中host这个tag为ceph-1的所有tag:

```
> show tag keys from ceph where host="ceph-1"
name: ceph
tagKey
------
collection
host
id
type
```


### SHOW TAG VALUES
显示tag values，命令格式为：

```
show_tag_values_stmt = "SHOW TAG VALUES" [on_clause] [ from_clause ] with_tag_clause [ where_clause ]
                       [ limit_clause ] [ offset_clause ] .
```

show tag values必须至少指定一个key的名称，可以查询某个或某几个key的数据库所有表中的tag value，也可以查询某个表中的tag values。


显示tag为host的所有值(如果不指定表，会列出所有表中tag的key为host的所有value)


```
> SHOW TAG VALUES WITH KEY = "host"
name: apache
key  value
---  -----
host openstack-1

name: ceph
key  value
---  -----
host ceph-1

name: cpu
key  value
---  -----
host 10.12.10.43
host ceph-1
......
```




# 数据管理
## ref
* [https://docs.influxdata.com/influxdb/v1.2/concepts/crosswalk/](https://docs.influxdata.com/influxdb/v1.2/concepts/crosswalk/)
* [https://docs.influxdata.com/influxdb/v1.2/query_language/data_exploration/](https://docs.influxdata.com/influxdb/v1.2/query_language/data_exploration/)

## 注意事项
* 不支持JOIN操作
* 如果标识符中包含除了[A-z,0-9,_]之外的字符串或者它们以数字开头，或者它们是InfluxDB的关键字，我们必须使用双引号括起来
* WHERE条件中的值不能使用双引号
* 每一个表中都有一个名为time的字段，可以在where中使用，比如**WHERE time > now() - 7d**
* group by后面可以使用tag key或者time interval，不能使用filed key
* select查询出来的都是epoch类型的时间戳，如果想看到的是字符串类型的时间，比如，可以在cli中使用**precision rfc3339**命令，切换时间显示格式。常用的格式包括(rfc3339, h, m, s, ms, u or ns).Unix时间戳的长度是10位整数(**date +%s** 命令可以查看当前时间戳)，InfluxDB默认的显示格式是ns，也就是19位整数.



## SELECT
用法:

```
> SELECT <field_key>[,<field_key>,<tag_key>] FROM <measurement_name>[,<measurement_name>]
```

注意：

* SELECT语句中可以使用field和tag，但是如果有tag，必须至少包含一个field
* 如果表中field和tag的名称相同，可以使用 **"filed_name"::field** 或者 **"tag_name"::tag** 来区分
* 如果只是在SELECT中指定tag key，不会报错，会返回空的数据


## WHERE
用法:

```
> SELECT_clause FROM_clause WHERE <conditional_expression> [(AND|OR) <conditional_expression> [...]]
```

注意事项:

* where条件中的字符串类型的值必须用单引号括起来；如果使用双引号，不会报错，也不会得到任何值。
* where条件中float类型的值一定不能使用单引号，否则查不到数据。

WHERE条件中支持对filed、tag和timestamp进行操作。

### field
格式:

```
field_key <operator> ['string' | boolean | float | integer]
```

支持的操作符:

```
=   equal to
<> not equal to
!= not equal to
>   greater than
>= greater than or equal to
<   less than
<= less than or equal to
```

### tag
格式:

```
tag_key <operator> ['tag_value']
```

支持的操作符:

```
=   equal to
<> not equal to
!= not equal to
```

### timestamps
InfluxDB中的所有时间都是UTC时间。

注意：

* 对于SELECT语句，默认的时间范围是 1677-09-21 00:12:43.145224194 到 2262-04-11T23:47:16.854775806Z。
* 对于带有GROUP BY time()的SELECT语句，默认的时间范围是1677-09-21 00:12:43.145224194 UTC 到 now()


### demo
查询CPU的load15大于10的数据(其中host是一个tag，代表主机名):

```
> select load15, host from system where "load15" > 10
name: system
time                load15 host
----                ------ ----
1489474580000000000 10.05  nl-cloud-openstack-1
1489474590000000000 10.07  nl-cloud-openstack-1
1489474600000000000 10.09  nl-cloud-openstack-1
1489474610000000000 10.14  nl-cloud-openstack-1
1489474620000000000 10.08  nl-cloud-openstack-1
```


## GROUP BY
* group by需要和聚合函数结合使用
* group by后面可以使用tag key或者time interval，不能使用filed key.


### group by tag key
格式:

```
> SELECT_clause FROM_clause [WHERE_clause] GROUP BY [* | <tag_key>[,<tag_key]]
```

如果想对所有的tag key做group by，可以使用GROUP BY *

比如想按照主机分组，查询CPU的load15的平均值

```
> select mean(load15) from system group by host
name: system
tags: host=nl-cloud-ceph-1
time mean
---- ----
0    0.09120040939667792

name: system
tags: host=nl-cloud-ceph-2
time mean
---- ----
0    0.11674878345502718
......
```

* 可以看到，group by之后，time的值是0.在InfluxDB中， epoch(时间点) 0 (1970-01-01T00:00:00Z) 一般表示空的时间戳。


### group by time intervals
格式:

```
> SELECT <function>(<field_key>) FROM_clause WHERE <time_range> GROUP BY time(<time_interval>),[tag_key] [fill(<fill_option>)]
```

查询'2017-03-23T00:00:00Z'到'2017-03-23T00:05:00Z'之间的数据:

```
> select load15 from system where host='nl-cloud-ceph-1' and time >= '2017-03-23T00:00:00Z' and time <= '2017-03-23T00:05:00Z'
name: system
time                 load15
----                 ------
2017-03-23T00:00:00Z 0.05
2017-03-23T00:00:10Z 0.05
2017-03-23T00:00:20Z 0.05
2017-03-23T00:00:30Z 0.05
2017-03-23T00:00:40Z 0.05
2017-03-23T00:00:50Z 0.05
2017-03-23T00:01:00Z 0.05
2017-03-23T00:01:10Z 0.05
2017-03-23T00:01:20Z 0.05
2017-03-23T00:01:30Z 0.05
2017-03-23T00:01:40Z 0.05
2017-03-23T00:01:50Z 0.05
2017-03-23T00:02:00Z 0.05
2017-03-23T00:02:10Z 0.05
2017-03-23T00:02:20Z 0.05
2017-03-23T00:02:30Z 0.05
2017-03-23T00:02:40Z 0.05
2017-03-23T00:02:50Z 0.05
2017-03-23T00:03:00Z 0.05
2017-03-23T00:03:10Z 0.05
2017-03-23T00:03:20Z 0.05
2017-03-23T00:03:30Z 0.05
2017-03-23T00:03:40Z 0.05
2017-03-23T00:03:50Z 0.05
2017-03-23T00:04:00Z 0.05
2017-03-23T00:04:10Z 0.05
2017-03-23T00:04:20Z 0.05
2017-03-23T00:04:30Z 0.05
2017-03-23T00:04:40Z 0.05
2017-03-23T00:04:50Z 0.05
2017-03-23T00:05:00Z 0.05
```

查询'2017-03-23T00:00:00Z'到'2017-03-23T00:05:00Z'之间的数据，按照1m进行分组，对每个分组求平均值

```
> select mean(load15) from system where host='nl-cloud-ceph-1' and time >= '2017-03-23T00:00:00Z' and time <= '2017-03-23T00:05:00Z' group by time(1m)
name: system
time                 mean
----                 ----
2017-03-23T00:00:00Z 0.049999999999999996
2017-03-23T00:01:00Z 0.049999999999999996
2017-03-23T00:02:00Z 0.049999999999999996
2017-03-23T00:03:00Z 0.049999999999999996
2017-03-23T00:04:00Z 0.049999999999999996
2017-03-23T00:05:00Z 0.05
```

### fill
fill是可选的，如果使用group by time(...)没有返回数据，使用什么方式来替换这个值。fill的选项包括:

* linear: 线性插入这个值
* none: 没有数据也没有time interval
* null: 没有数据，有time interval。这是默认的行为
* previous: 如果有time interval没有数据，使用上一个数据


> 注意: fill必须在group by子句的最后面


# 函数
[https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#aggregations](https://docs.influxdata.com/influxdb/v1.2/query_language/functions/#aggregations)


## Aggregations(聚合函数)
聚合函数只能使用field key，而不能使用tag key作为参数。

### COUNT
COUNT函数的功能是返回某个字段非空值的个数。

比如我想查看system表中，load1这个字段一共有多少个非空的值:

```
> select count(load1) from system;
name: system
time count
---- -----
0    2490856
```

可以看到一共有2490856个非空值


比如我想查看system表中，load1这个字段一共有多少个非空的值，并且按照host这个tag进行分组:

```
> select count(load1) from system group by host;
name: system
tags: host=nl-cloud-ceph-1
time count
---- -----
0    81126

name: system
tags: host=nl-cloud-ceph-2
time count
---- -----
0    81254

name: system
tags: host=nl-cloud-ceph-3
time count
---- -----
0    51619
......
```


### DISTINCT
DISTINCT返回一个字段不重复的值的列表


### INTEGRAL
1.2版本中未实现。


### MEAN
返回某个字段的平均值，字段必须是int64或float64的。

格式:

```
> SELECT MEAN(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

返回过去一天内nl-cloud-ceph-1主机load15的平均值

```
> select mean(load15) from system where host='nl-cloud-ceph-1' and time >= now() - 1d
name: system
time                           mean
----                           ----
2017-03-26T07:06:24.945676036Z 0.07806597222222564
```


### MEDIAN
median是中位数。

定义引用自wiki: 对于一组有限个数的数据来说，它们的中位数是这样的一种数：这群数据里的一半的数据比它大，而另外一半数据比它小。 计算有限个数的数据的中位数的方法是：把所有的同类数据按照大小的顺序排列。 如果数据的个数是奇数，则中间那个数据就是这群数据的中位数；如果数据的个数是偶数，则中间那2个数据的算术平均值就是这群数据的中位数。

```
> select median(load15) from system where host='nl-cloud-ceph-1' and time >= now() - 10d
name: system
time                           median
----                           ------
2017-03-17T07:25:54.355346846Z 0.06
```


### MODE
返回某个key出现频率最高的value

```
> select mode(tcp_currestab) from net where host='nl-cloud-ceph-1' and time >= now() - 10d
name: net
time                           mode
----                           ----
2017-03-17T07:30:18.948334757Z 242
```

表示242在tcp_currestab这个filed中出现的频率最高

### SPREAD
计算最大值和最小值之间的差距

比如计算一天内最大连接数和最小连接数之间的差异:

```
> select spread(tcp_currestab) from net where host='nl-cloud-ceph-1' and time >= now() - 1d
name: net
time                           spread
----                           ------
2017-03-26T07:42:07.905201326Z 6
```

通过下面的max可以看到最大是192，min可以看到最小是186，相减正好是6.

也可以按照6小时做一次分组，那么可以得到24/6+1=5组数据:

```
> select spread(tcp_currestab) from net where host='nl-cloud-ceph-1' and time >= now() - 1d group by time(6h)
name: net
time                 spread
----                 ------
2017-03-26T06:00:00Z 2
2017-03-26T12:00:00Z 1
2017-03-26T18:00:00Z 1
2017-03-27T00:00:00Z 4
2017-03-27T06:00:00Z 5
```


### STDDEV
标准差是方差(平方差)的平方根，反应的是数据离散的程度，越大数据之间的差异越大。

```
> select stddev(tcp_currestab) from net where host='nl-cloud-ceph-1' and time >= now() - 1d group by time(6h)
name: net
time                 stddev
----                 ------
2017-03-26T06:00:00Z 0.23537934230121352
2017-03-26T12:00:00Z 0.08838576993946781
2017-03-26T18:00:00Z 0.23267329099713924
2017-03-27T00:00:00Z 0.18383302430318563
2017-03-27T06:00:00Z 0.6003860353212642
```

### SUM
求和


## Selectors(选择函数)
### FIRST
获取第一个值

返回过去一天内nl-cloud-ceph-1主机已建立网络连接数的第一个值

```
> select first(tcp_currestab) from net where host='nl-cloud-ceph-1' and time >= now() - 1d
name: net
time                 first
----                 -----
2017-03-26T07:35:30Z 187
```


### LAST
获取最后一个值

返回过去一天内nl-cloud-ceph-1主机已建立网络连接数的最后一个值

```
> select last(tcp_currestab) from net where host='nl-cloud-ceph-1' and time >= now() - 1d
name: net
time                 last
----                 ----
2017-03-27T07:33:40Z 187
```


### MAX
获取最大值

返回过去一天内nl-cloud-ceph-1主机已建立网络连接数的最大值

```
> select max(tcp_currestab) from net where host='nl-cloud-ceph-1' and time >= now() - 1d
name: net
time                 max
----                 ---
2017-03-27T06:50:30Z 192
```


### MIN
获取最大值

返回过去一天内nl-cloud-ceph-1主机已建立网络连接数的最小值

```
> select min(tcp_currestab) from net where host='nl-cloud-ceph-1' and time >= now() - 1d
name: net
time                 min
----                 ---
2017-03-26T07:47:20Z 186
```


### TOP
获取最大的N个值


返回过去一天内nl-cloud-ceph-1主机已建立网络连接数的的最大的3个值

```
> select top(tcp_currestab, 3) from net where host='nl-cloud-ceph-1' and time >= now() - 1d
name: net
time                 top
----                 ---
2017-03-27T06:50:30Z 192
2017-03-27T06:50:40Z 192
2017-03-27T06:50:50Z 192
```

### BOTTOM
获取最小的N个值

返回过去一天内nl-cloud-ceph-1主机已建立网络连接数的的最小的3个值

```
> select bottom(tcp_currestab, 3) from net where host='nl-cloud-ceph-1' and time >= now() - 1d
name: net
time                 bottom
----                 ------
2017-03-26T07:47:20Z 186
2017-03-26T07:47:30Z 186
2017-03-26T07:47:40Z 186
```


### PERCENTILE
格式: PERCENTILE(key, N)

PERCENTILE可以返回key对应value前N%的值(N的取值范围是0-100):

* PERCENTILE(<field_key>,100) 就是MAX(<field_key>)
* PERCENTILE(<field_key>,0) 就是MIN(<field_key>)
* PERCENTILE(<field_key>,50) 就是MEDIAN(<field_key>)

比如，查询CPU的load1中大于80%其他值的值是多少。

返回过去一天内nl-cloud-ceph-1主机已建立网络连接数中大于80%其他值的值是多少。

```
> select percentile(tcp_currestab, 80) from net where host='nl-cloud-ceph-1' and time >= now() - 1d
name: net
time                 percentile
----                 ----------
2017-03-27T02:50:30Z 188
```

查询出来的结果是188，表示188会大于查询的数据集中80%的值。


### SAMPLE
随机返回一些数据点

> SELECT SAMPLE(<field_key>,<N>) FROM_clause [WHERE_clause] [GROUP_BY_clause]

```
> select sample(load1, 3) from system where host='nl-cloud-openstack-1' and time >= now() - 1d
name: system
time                 sample
----                 ------
2017-03-26T10:53:10Z 0.45
2017-03-26T13:52:00Z 0.32
2017-03-26T16:27:30Z 0.07
```


## Transformations(转换函数)
### CEILING
1.2版本中暂未实现。

### CUMULATIVE_SUM
累加和。


求平均值：

```
> select mean(tcp_currestab) from net where host='nl-cloud-openstack-1' and time >= now() - 1d group by time(12h)
name: net
time                 mean
----                 ----
2017-03-26T00:00:00Z 143.7172131147541
2017-03-26T12:00:00Z 143.41736111111112
2017-03-27T00:00:00Z 143.2752100840336
```

对平均值做累加(自己的值加上个组的值)

```
> select cumulative_sum(mean(tcp_currestab)) from net where host='nl-cloud-openstack-1' and time >= now() - 1d group by time(12h)
name: net
time                 cumulative_sum
----                 --------------
2017-03-26T00:00:00Z 143.71692940370116
2017-03-26T12:00:00Z 287.1342905148123
2017-03-27T00:00:00Z 430.40832057423205
```

### DERIVATIVE
衍生，返回数据的变化率。(可以用来计算pps、网速、磁盘读写速率).

> SELECT DERIVATIVE(<field_key>, [<unit>]) FROM <measurement_name> [WHERE <stuff>]

derivative函数计算一个字段上两个数据之间的差值，然后除以unit(如果不指定，默认是1s)。

查询网卡的包量:

```
> SELECT packets_recv FROM "net" where host='nl-cloud-openstack-1' and time >= now() - 1m
name: net
time                 packets_recv
----                 ------------
2017-03-27T08:26:50Z 82409624
2017-03-27T08:27:00Z 82409767
2017-03-27T08:27:10Z 82409923
2017-03-27T08:27:20Z 82410066
2017-03-27T08:27:30Z 82410212
2017-03-27T08:27:40Z 82410352
```

我们用data_interval表示两个数据之间的时间差，比如上面的时间差是10s


进行derivative操作:

```
> SELECT derivative("packets_recv", 1s) FROM "net" where host='nl-cloud-openstack-1' and time >= now() - 1m
name: net
time                 derivative
----                 ----------
2017-03-27T08:27:00Z 14.3
2017-03-27T08:27:10Z 15.6
2017-03-27T08:27:20Z 14.3
2017-03-27T08:27:30Z 14.6
2017-03-27T08:27:40Z 14
```

我们用derivative_unit表示derivative中的单位(默认是1s)，当前时间derivative值的计算公式是:

> 当前时间derivative值=(当前时间的值 - 上一个时间的值)/(data_interval/derivative_unit)。

我们以2017-03-27T08:27:00Z这个时间的数据为例，当前的值是82409767，上一个时间2017-03-27T08:26:50Z的值是82409624，data_interval为两个点之间的时间差(2017-03-27T08:27:00Z - 2017-03-27T08:26:50Z = 10s)，derivative_unit为1s，根据计算公式:

> (82409767 - 82409624)/( 10s / 1s ) = 14.3


如果使用group by time(interval)


```
> SELECT derivative("packets_recv", 10s) FROM "net" where host='nl-cloud-openstack-1' and time >= now() - 1m
name: net
time                 derivative
----                 ----------
2017-03-27T08:27:10Z 156
2017-03-27T08:27:20Z 143
2017-03-27T08:27:30Z 146
2017-03-27T08:27:40Z 140
2017-03-27T08:27:50Z 149
```



### DIFFERENCE
求差。

计算每天的最小连接数值：

```
> select min(tcp_currestab) from net where host='nl-cloud-openstack-1' and time >= now() - 10d group by time(1d)
name: net
time                 min
----                 ---
2017-03-17T00:00:00Z 158
2017-03-18T00:00:00Z 158
2017-03-19T00:00:00Z 158
2017-03-20T00:00:00Z 156
2017-03-21T00:00:00Z 137
2017-03-22T00:00:00Z 137
2017-03-23T00:00:00Z 137
2017-03-24T00:00:00Z 141
2017-03-25T00:00:00Z 141
2017-03-26T00:00:00Z 141
2017-03-27T00:00:00Z 141
```

计算每天最小连接数的变化:

```
> select difference(min(tcp_currestab)) from net where host='nl-cloud-openstack-1' and time >= now() - 10d group by time(1d)
name: net
time                 difference
----                 ----------
2017-03-18T00:00:00Z 0
2017-03-19T00:00:00Z 0
2017-03-20T00:00:00Z -2
2017-03-21T00:00:00Z -19
2017-03-22T00:00:00Z 0
2017-03-23T00:00:00Z 0
2017-03-24T00:00:00Z 4
2017-03-25T00:00:00Z 0
2017-03-26T00:00:00Z 0
2017-03-27T00:00:00Z 0
```


### ELAPSED
单位:

```
> SELECT ELAPSED(<field_key>, <unit>) FROM <measurement_name> [WHERE <stuff>]
```

显示两个数据点之间的经历的时间差的个数，默认单位为ns(纳秒)。比如:

```
> select elapsed(tcp_currestab) from net where host='nl-cloud-openstack-1' and time >= now() - 1m
name: net
time                 elapsed
----                 -------
2017-03-27T08:14:30Z 10000000000
2017-03-27T08:14:40Z 10000000000
2017-03-27T08:14:50Z 10000000000
2017-03-27T08:15:00Z 10000000000
2017-03-27T08:15:10Z 10000000000
```

以5s为单位显示：

```
> select elapsed(tcp_currestab, 5s) from net where host='nl-cloud-openstack-1' and time >= now() - 1m
name: net
time                 elapsed
----                 -------
2017-03-27T08:20:20Z 2
2017-03-27T08:20:30Z 2
2017-03-27T08:20:40Z 2
2017-03-27T08:20:50Z 2
2017-03-27T08:21:00Z 2
```

由于两个数据之间的时间间隔是10s，单位是5s，所以计算出来的是2。

注意，如果unit大于数据之间的时间间隔，会显示为0，比如以20s为单位显示：

```
> select elapsed(tcp_currestab, 20s) from net where host='nl-cloud-openstack-1' and time >= now() - 1m
name: net
time                 elapsed
----                 -------
2017-03-27T08:17:50Z 0
2017-03-27T08:18:00Z 0
2017-03-27T08:18:10Z 0
2017-03-27T08:18:20Z 0
```

### FLOOR
1.2版本中暂未实现。

### HISTOGRAM
1.2版本中暂未实现。

### MOVING_AVERAGE

### NON_NEGATIVE_DERIVATIVE

## Predictors(预测函数)
### HOLT_WINTERS

