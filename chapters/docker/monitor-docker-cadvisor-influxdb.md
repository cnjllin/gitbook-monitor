# cAdvisor+InfluxDB+Grafana集成

* InfluxDB的安装配置参考[这里](../basic/monitor-basic-influxdb.html)
* Grafana安装与配置参考[这里](../basic/monitor-basic-grafana.html)
* InfluxDB和Grafana的集成参考[这里](../basic/monitor-basic-grafana-tutorial.html)

下面主要演示cAdvisor收集的数据怎么写入到InfluxDB中。


参考: [https://www.brianchristner.io/how-to-setup-docker-monitoring/](https://www.brianchristner.io/how-to-setup-docker-monitoring/)

启动容器时加上InfluxDB相关的参数:

```
# docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw  \
    --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro
    --publish=8080:8080 --detach=true --link influxsrv:influxsrv --name=cadvisor \
    google/cadvisor:latest
    -storage_driver_db=cadvisor
    -storage_driver_host=IP:8086
    -storage_driver_user=cadvisor
    -storage_driver_password=password
```


发现一个坑,cAdvisor连接InfluxDB时,虽然可以配置用户名和密码,但是没有TLS相关的配置(kafka倒是有)。 cadvisor的配置如下:

```
/ # cadvisor  -h
Usage of cadvisor:
  -allow_dynamic_housekeeping
    	Whether to allow the housekeeping interval to be dynamic (default true)
  -alsologtostderr
    	log to standard error as well as files
  -application_metrics_count_limit int
    	Max number of application metrics to store (per container) (default 100)
  -boot_id_file string
    	Comma-separated list of files to check for boot-id. Use the first one that exists. (default "/proc/sys/kernel/random/boot_id")
  -bq_account string
    	Service account email
  -bq_credentials_file string
    	Credential Key file (pem)
  -bq_id string
    	Client ID
  -bq_project_id string
    	Bigquery project ID
  -bq_secret string
    	Client Secret (default "notasecret")
  -collector_cert string
    	Collector's certificate, exposed to endpoints for certificate based authentication.
  -collector_key string
    	Key for the collector's certificate
  -container_hints string
    	location of the container hints file (default "/etc/cadvisor/container_hints.json")
  -disable_metrics metrics
    	comma-separated list of metrics to be disabled. Options are 'disk', 'network', 'tcp'. Note: tcp is disabled by default due to high CPU usage. (default tcp)
  -docker string
    	docker endpoint (default "unix:///var/run/docker.sock")
  -docker_env_metadata_whitelist string
    	a comma-separated list of environment variable keys that needs to be collected for docker containers
  -docker_only
    	Only report docker containers in addition to root stats
  -docker_root string
    	DEPRECATED: docker root is read from docker info (this is a fallback, default: /var/lib/docker) (default "/var/lib/docker")
  -enable_load_reader
    	Whether to enable cpu load reader
  -event_storage_age_limit string
    	Max length of time for which to store events (per type). Value is a comma separated list of key values, where the keys are event types (e.g.: creation, oom) or "default" and the value is a duration. Default is applied to all non-specified event types (default "default=24h")
  -event_storage_event_limit string
    	Max number of events to store (per type). Value is a comma separated list of key values, where the keys are event types (e.g.: creation, oom) or "default" and the value is an integer. Default is applied to all non-specified event types (default "default=100000")
  -global_housekeeping_interval duration
    	Interval between global housekeepings (default 1m0s)
  -housekeeping_interval duration
    	Interval between container housekeepings (default 1s)
  -http_auth_file string
    	HTTP auth file for the web UI
  -http_auth_realm string
    	HTTP auth realm for the web UI (default "localhost")
  -http_digest_file string
    	HTTP digest file for the web UI
  -http_digest_realm string
    	HTTP digest file for the web UI (default "localhost")
  -listen_ip string
    	IP to listen on, defaults to all IPs
  -log_backtrace_at value
    	when logging hits line file:N, emit a stack trace
  -log_cadvisor_usage
    	Whether to log the usage of the cAdvisor container
  -log_dir string
    	If non-empty, write log files in this directory
  -logtostderr
    	log to standard error instead of files
  -machine_id_file string
    	Comma-separated list of files to check for machine-id. Use the first one that exists. (default "/etc/machine-id,/var/lib/dbus/machine-id")
  -max_housekeeping_interval duration
    	Largest interval to allow between container housekeepings (default 1m0s)
  -max_procs int
    	max number of CPUs that can be used simultaneously. Less than 1 for default (number of cores).
  -port int
    	port to listen (default 8080)
  -profiling
    	Enable profiling via web interface host:port/debug/pprof/
  -prometheus_endpoint string
    	Endpoint to expose Prometheus metrics on (default "/metrics")
  -stderrthreshold value
    	logs at or above this threshold go to stderr
  -storage_driver driver
    	Storage driver to use. Data is always cached shortly in memory, this controls where data is pushed besides the local cache. Empty means none. Options are: <empty>, bigquery, elasticsearch, influxdb, kafka, redis, statsd, stdout
  -storage_driver_buffer_duration duration
    	Writes in the storage driver will be buffered for this duration, and committed to the non memory backends as a single transaction (default 1m0s)
  -storage_driver_db string
    	database name (default "cadvisor")
  -storage_driver_es_enable_sniffer
    	ElasticSearch uses a sniffing process to find all nodes of your cluster by default, automatically
  -storage_driver_es_host string
    	ElasticSearch host:port (default "http://localhost:9200")
  -storage_driver_es_index string
    	ElasticSearch index name (default "cadvisor")
  -storage_driver_es_type string
    	ElasticSearch type name (default "stats")
  -storage_driver_host string
    	database host:port (default "localhost:8086")
  -storage_driver_kafka_broker_list string
    	kafka broker(s) csv (default "localhost:9092")
  -storage_driver_kafka_ssl_ca string
    	optional certificate authority file for TLS client authentication
  -storage_driver_kafka_ssl_cert string
    	optional certificate file for TLS client authentication
  -storage_driver_kafka_ssl_key string
    	optional key file for TLS client authentication
  -storage_driver_kafka_ssl_verify
    	verify ssl certificate chain (default true)
  -storage_driver_kafka_topic string
    	kafka topic (default "stats")
  -storage_driver_password string
    	database password (default "root")
  -storage_driver_secure
    	use secure connection with database
  -storage_driver_table string
    	table name (default "stats")
  -storage_driver_user string
    	database username (default "root")
  -storage_duration duration
    	How long to keep data stored (Default: 2min). (default 2m0s)
  -v value
    	log level for V logs
  -version
    	print cAdvisor version and exit
  -vmodule value
    	comma-separated list of pattern=N settings for file-filtered logging
```

