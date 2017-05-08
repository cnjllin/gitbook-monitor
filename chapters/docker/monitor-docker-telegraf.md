# Telegraf

使用telegraf的docker input plugin监控Docker

## 配置
编辑telegraf的配置文件,将inputs.docker部分取消注释:

```
[[inputs.docker]]
  ## Docker Endpoint
  ##   To use TCP, set endpoint = "tcp://[ip]:[port]"
  ##   To use environment variables (ie, docker-machine), set endpoint = "ENV"
  endpoint = "unix:///var/run/docker.sock"
  ## Only collect metrics for these containers, collect all if empty
  container_names = []
  ## Timeout for docker list, info, and stats commands
  timeout = "5s"

  ## Whether to report for each container per-device blkio (8:0, 8:1...) and
  ## network (eth0, eth1, ...) stats or not
  perdevice = true
  ## Whether to report for each container total blkio and network stats or not
  total = false
```


碰到的问题

```
# journalctl -lf -u telegraf
May 07 17:55:20 flannel-1 telegraf[13820]: Cannot connect to the Docker daemon. Is the docker daemon running on this host?
May 07 17:55:20 flannel-1 telegraf[13820]: 2017-05-07T17:55:20Z E! ERROR in input [inputs.docker]: Cannot connect to the Docker daemon. Is the docker daemon running on this host?
......
```


使用root用户可以获取数据:

```
# telegraf telegraf -test -input-filter=docker
......
```


Google了一下,可能是telegraf用户没有权限导致的。

切换到telegraf用户测试收集数据,发现:

```
# sudo -u telegraf telegraf -test -input-filter=docker
* Plugin: inputs.docker, Collection 1
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
......

```

网上有种解决方案是将telegraf用户加入到docker组,但是我的宿主机上没有docker组,可能是版本不一样。


另一种解决方法是,修改/usr/lib/systemd/system/telegraf.service文件,将telegraf进程的User设置为root,如下:

```
...
[Service]
EnvironmentFile=-/etc/default/telegraf
User=root
...
```

重新加载service文件并重新启动telegraf,查看日志就没有错误了。

```
# systemctl daemon-reload
# systemctl restart telegraf
# journalctl -lf -u telegraf
```

## InfluxDB表结构
配置好后,登录到InfluxDB中,可以看到多了几张关于Docker的measurement:

```
> use telegraf
Using database telegraf
> show measurements
name: measurements
name
----
apache
ceph
cpu
disk
diskio
docker
docker_container_blkio
docker_container_cpu
docker_container_mem
docker_container_net
docker_data
docker_metadata
kernel
```


可以看到docker相关的表有7个:

* docker
* docker_container_blkio
* docker_container_cpu
* docker_container_mem
* docker_container_net
* docker_data
* docker_metadata



### docker
```
> show field keys on telegraf from docker
name: docker
fieldKey                fieldType
--------                ---------
memory_total            integer
n_containers            integer
n_containers_paused     integer
n_containers_running    integer
n_containers_stopped    integer
n_cpus                  integer
n_goroutines            integer
n_images                integer
n_listener_events       integer
n_used_file_descriptors integer
pool_blocksize          integer
```

这个measurement的信息主要和docker info命令类似。  

* n_images: 宿主机上镜像的个数
* n_containers: 容器的总个数
* n_containers_running: 处于运行状态的容器的总个数
* n_containers_stopped: 处于停止状态的容器的总个数
* n_containers_paused: 处于暂停状态的容器的总个数
* memory_total: 宿主机的物理内存总量
* n_cpus: 宿主机逻辑CPU的个数
* n_used_file_descriptors: 使用的文件描述符的个数

### docker_data
```
> show field keys on telegraf from docker_data
name: docker_data
fieldKey  fieldType
--------  ---------
available integer
total     integer
used      integer
```

* available: 对应docker info中的"Data Space Available"
* total: 对应docker info中的"Data Space Total"
* used: 对应docker info中的"Data Space Used" 



docker_data的单位是GB，在Grafana中的数据单位要选择data(Metric)->bits，而不是data(ICE)->bits

1KB = 1,000 Byte
1KiB = 1,024Byte



### docker_metadata
```
> show field keys on telegraf from docker_metadata
name: docker_metadata
fieldKey  fieldType
--------  ---------
available integer
total     integer
used      integer
```

* available: 对应docker info中的"Metadata Space Available"
* total: 对应docker info中的"Metadata Space Total"
* used: 对应docker info中的"Metadata Space Used" 



### docker_container_cpu
```
> show field keys on telegraf from docker_container_cpu
name: docker_container_cpu
fieldKey                     fieldType
--------                     ---------
container_id                 string
throttling_periods           integer
throttling_throttled_periods integer
throttling_throttled_time    integer
usage_in_kernelmode          integer
usage_in_usermode            integer
usage_percent                float
usage_system                 integer
usage_total                  integer
```

* container_id:
* throttling_periods: 
* throttling_throttled_periods: 
* throttling_throttled_time: 
* usage_in_kernelmode: 
* usage_in_usermode: 
* usage_percent: 
* usage_system: 
* usage_total: 




### docker_container_mem
```
> show field keys on telegraf from docker_container_mem
name: docker_container_mem
fieldKey                  fieldType
--------                  ---------
active_anon               integer
active_file               integer
cache                     integer
container_id              string
fail_count                integer
hierarchical_memory_limit integer
inactive_anon             integer
inactive_file             integer
limit                     integer
mapped_file               integer
max_usage                 integer
pgfault                   integer
pgmajfault                integer
pgpgin                    integer
pgpgout                   integer
rss                       integer
rss_huge                  integer
total_active_anon         integer
total_active_file         integer
total_cache               integer
total_inactive_anon       integer
total_inactive_file       integer
total_mapped_file         integer
total_pgfault             integer
total_pgmafault           integer
total_pgpgin              integer
total_pgpgout             integer
total_rss                 integer
total_rss_huge            integer
total_unevictable         integer
total_writeback           integer
unevictable               integer
usage                     integer
usage_percent             float
writeback                 integer
```



### docker_container_net
```
> show field keys on telegraf from docker_container_net
name: docker_container_net
fieldKey     fieldType
--------     ---------
container_id string
rx_bytes     integer
rx_dropped   integer
rx_errors    integer
rx_packets   integer
tx_bytes     integer
tx_dropped   integer
tx_errors    integer
tx_packets   integer
```

* container_id: 容器id
* rx_packets: recieve, 接受到的packet的个数
* rx_bytes: 接受到的数据的byte数
* rx_dropped: 接受时drop掉的
* rx_errors: 接受时错误的


### docker_container_blkio
```
> show field keys on telegraf from docker_container_blkio
name: docker_container_blkio
fieldKey                         fieldType
--------                         ---------
container_id                     string
io_service_bytes_recursive_async integer
io_service_bytes_recursive_read  integer
io_service_bytes_recursive_sync  integer
io_service_bytes_recursive_total integer
io_service_bytes_recursive_write integer
io_serviced_recursive_async      integer
io_serviced_recursive_read       integer
io_serviced_recursive_sync       integer
io_serviced_recursive_total      integer
io_serviced_recursive_write      integer
```


## Grafana配置
### cpu利用率图

### 内存使用图

### blkio图

### 带宽图

### pps图






