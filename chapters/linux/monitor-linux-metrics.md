# Linux监控

本文主要介绍了Telegraf中Linux相关的监控指标。

查看监控项分类(查询telegraf表中的measurement):

```
> use telegraf
> show measurements
name: measurements
name
----
cpu
disk
diskio
kernel
mem
net
processes
swap
system
```

## cpu
查看cpu表中的metrics:  

```
> show field keys on telegraf from cpu
name: cpu
fieldKey         fieldType
--------         ---------
usage_guest      float
usage_guest_nice float
usage_idle       float
usage_iowait     float
usage_irq        float
usage_nice       float
usage_softirq    float
usage_steal      float
usage_system     float
usage_user       float
```

如下:  

* usage_user: 用户态占用的CPU时间百分比
* usage_system: 内核态占用的CPU时间百分比
* usage_iowait: io等待占用的CPU时间百分比
* usage_idle: CPU空闲时间百分比
* usage_irq: 硬中断占用的CPU时间百分比
* usage_softirq: 软中断占用的CPU时间百分比
* usage_nice: 被nice的进程占用的CPU时间百分比
* usage_steal: 虚拟机占用的CPU时间百分比(如果OS作为Hypervisor)


## processes
查看processes表中的metrics:

```
> show field keys on telegraf from processes
name: processes
fieldKey      fieldType
--------      ---------
blocked       integer
paging        integer
running       integer
sleeping      integer
stopped       integer
total         integer
total_threads integer
unknown       integer
zombies       integer
```


## system
查看system表中的metrics:

```
> show field keys on telegraf from system
name: system
fieldKey      fieldType
--------      ---------
load1         float
load15        float
load5         float
n_cpus        integer
n_users       integer
uptime        integer
uptime_format string
```


## kernel
查看kernel表中的metrics:

```
> show field keys on telegraf from kernel
name: kernel
fieldKey         fieldType
--------         ---------
boot_time        integer
context_switches integer
interrupts       integer
processes_forked integer
```

## mem
查看mem表中的metrics:

```
> show field keys on telegraf from mem
name: mem
fieldKey          fieldType
--------          ---------
active            integer
available         integer
available_percent float
buffered          integer
cached            integer
free              integer
inactive          integer
total             integer
used              integer
used_percent      float
```




## swap
查看swap表中的metrics:

```
> show field keys on telegraf from swap
name: swap
fieldKey     fieldType
--------     ---------
free         integer
in           integer
out          integer
total        integer
used         integer
used_percent float
```


## disk
查看disk表中的metrics:  

```
> show field keys on telegraf from disk
name: disk
fieldKey     fieldType
--------     ---------
free         integer
inodes_free  integer
inodes_total integer
inodes_used  integer
total        integer
used         integer
used_percent float
```

* free:  
* inodes_free:  
* inodes_total:  
* inodes_used:  
* total:  
* used:  
* used_percent:  



## diskio
查看diskio表中的metrics:

```
> show field keys on telegraf from diskio
name: diskio
fieldKey         fieldType
--------         ---------
io_time          integer
iops_in_progress integer
read_bytes       integer
read_time        integer
reads            integer
write_bytes      integer
write_time       integer
writes           integer
```



## net
查看net表中的metrics:

```
> show field keys on telegraf from net
name: net
fieldKey              fieldType
--------              ---------
bytes_recv            integer
bytes_sent            integer
drop_in               integer
drop_out              integer
err_in                integer
err_out               integer
icmp_inaddrmaskreps   integer
icmp_inaddrmasks      integer
icmp_incsumerrors     integer
icmp_indestunreachs   integer
icmp_inechoreps       integer
icmp_inechos          integer
icmp_inerrors         integer
icmp_inmsgs           integer
icmp_inparmprobs      integer
icmp_inredirects      integer
icmp_insrcquenchs     integer
icmp_intimeexcds      integer
icmp_intimestampreps  integer
icmp_intimestamps     integer
icmp_outaddrmaskreps  integer
icmp_outaddrmasks     integer
icmp_outdestunreachs  integer
icmp_outechoreps      integer
icmp_outechos         integer
icmp_outerrors        integer
icmp_outmsgs          integer
icmp_outparmprobs     integer
icmp_outredirects     integer
icmp_outsrcquenchs    integer
icmp_outtimeexcds     integer
icmp_outtimestampreps integer
icmp_outtimestamps    integer
icmpmsg_intype0       integer
icmpmsg_outtype3      integer
icmpmsg_outtype8      integer
ip_defaultttl         integer
ip_forwarding         integer
ip_forwdatagrams      integer
ip_fragcreates        integer
ip_fragfails          integer
ip_fragoks            integer
ip_inaddrerrors       integer
ip_indelivers         integer
ip_indiscards         integer
ip_inhdrerrors        integer
ip_inreceives         integer
ip_inunknownprotos    integer
ip_outdiscards        integer
ip_outnoroutes        integer
ip_outrequests        integer
ip_reasmfails         integer
ip_reasmoks           integer
ip_reasmreqds         integer
ip_reasmtimeout       integer
packets_recv          integer
packets_sent          integer
tcp_activeopens       integer
tcp_attemptfails      integer
tcp_currestab         integer
tcp_estabresets       integer
tcp_incsumerrors      integer
tcp_inerrs            integer
tcp_insegs            integer
tcp_maxconn           integer
tcp_outrsts           integer
tcp_outsegs           integer
tcp_passiveopens      integer
tcp_retranssegs       integer
tcp_rtoalgorithm      integer
tcp_rtomax            integer
tcp_rtomin            integer
udp_incsumerrors      integer
udp_indatagrams       integer
udp_inerrors          integer
udp_noports           integer
udp_outdatagrams      integer
udp_rcvbuferrors      integer
udp_sndbuferrors      integer
udplite_incsumerrors  integer
udplite_indatagrams   integer
udplite_inerrors      integer
udplite_noports       integer
udplite_outdatagrams  integer
udplite_rcvbuferrors  integer
udplite_sndbuferrors  integer
```






