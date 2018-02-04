# Grafana 配置



# 配置
http://docs.grafana.org/installation/configuration/

## SMTP配置
[http://docs.grafana.org/installation/configuration/#smtp](http://docs.grafana.org/installation/configuration/#smtp)

smtp部分配置如下:

```
[smtp]
enabled = true
host = smtp.vip.sina.com:25
user = dyc_abc
password = ...
;cert_file =
;key_file =
;skip_verify = false
from_address = dyc_abc@vip.sina.com
```

修改完配置文件后需要重启grafana

> systemctl restart grafana

### 异常
测试发送邮件时,发现发送失败,查看/var/log/grafana/grafana.log文件存在以下问题

> Notification service failed to initialize" logger=server erro="Invalid email address for smpt from_adress config

解决: 我的from_address配置的是**admin@grafana.10.12.10.8**,不是一个正常的email地址,修改为admin@grafana_10_12_10_8.com


> error="gomail: could not send email 1: 553 Envolope sender mismatch with login user.."

解决: user字段必须和from_address字段中@前面的一样


# 添加用户
添加只读权限的用户

Admin - Users - 'Add or Invite' - 用户选择Viewer - 点击完后生成一个邀请链接, 在浏览器中打开链接, 设置密码就可以添加用户了

Grafana权限:

* Editor: Can create & edit dashboads
* Viewer: Can not create dashboards nor edit panels
* Read only editor: Can not create dashboards nor save changes to them, but they can edit panels and create new panels (But not save)


# 通知
"Alerting" - "Notifications" - "New Notification"

**Send on all alerts**表示对所有的告警都使用这个通知。

设置完后,可以使用"Send Test"测试一下。

## email
smtp的配置参见上面。

Type设置为email,如果有多个地址,使用分号";"分隔。

## slack
### 申请incoming webhook
1. 点击左上角账户,点击 team - "Apps & integrations",搜索webhook(这个搜索功能很好用,还可以搜索token)
2. 找到"incoming webhook"然后点击进去
3. 点击左边的Request, 批注后配置一下就会生成一个url


### 配置
Type设置为slack。

* Url: 设置为上面申请的incoming webhook
* Recipient(接受者): 如果是某个人,使用@user;如果是某个chanel,使用#chanel
* Mention: 如果Recipient设置的是chanel,这个选项表示在这个chanel里面艾特某个具体的人,使用@user。





