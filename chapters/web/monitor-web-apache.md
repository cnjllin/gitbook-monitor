# apache/httpd监控
apache需要开启server-status模块，才能监控，配置参考()

telegraf将apache的监控数据存储在telegraf库的apache表中，如下:  

```
> show field keys on telegraf from apache
name: apache
fieldKey             fieldType
--------             ---------
BusyWorkers          float
BytesPerReq          float
BytesPerSec          float
CPULoad              float
IdleWorkers          float
ReqPerSec            float
TotalAccesses        float
TotalkBytes          float
Uptime               float
scboard_closing      float
scboard_dnslookup    float
scboard_finishing    float
scboard_idle_cleanup float
scboard_keepalive    float
scboard_logging      float
scboard_open         float
scboard_reading      float
scboard_sending      float
scboard_starting     float
scboard_waiting      float
```

BusyWorkers+IdleWorkers = ps -f -u apache | wc -l


怎么判断是否是busy






