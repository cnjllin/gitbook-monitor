# grafana使用域登录


修改配置文件

```
#################################### Auth LDAP ##########################
[auth.ldap]
;enabled = false
;config_file = /etc/grafana/ldap.toml
;allow_sign_up = true

enabled = true
config_file = /etc/grafana/ldap.toml
```


重启服务

```
systemctl restart grafana-server
systemctl status grafana-server
```



