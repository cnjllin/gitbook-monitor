# nginx监控

```
# curl localhost/status
Active connections: 1
server accepts handled requests
 2909 2909 2906
Reading: 0 Writing: 1 Waiting: 0
```

* Active connections: 活跃的连接数量
* accepts(2909): 接收到的客户端发来的连接数
* handled(2909): 已经处理完的连接数, 如果小于accepts很多说明现在nginx性能出现瓶颈
* requests(2906): 客户端请求总数
* Reading: 正在读取请求头信息的连接数
* Writing: 正在发送响应报文的连接数
* Waiting: 处于闲置状态正等待客户端发送请求的连接数。 开启keep-alive的情况下,这个值等于active – (reading+writing), 意思就是Nginx已经处理完正在等候下一次请求指令的驻留连接.

参考: https://www.jianshu.com/p/2adefc5efece







指标

* QPS: 每秒请求数
* 响应时间
* 服务器错误率


Requests per second
Response times
Active connections
Connection backlog queue
Response codes
Process open file handles
Process states
Server status
Server load average
Server network usage
Server disk space
and more!

