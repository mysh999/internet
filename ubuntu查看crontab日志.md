修改rsyslog

```bash
$ sudo vim /etc/rsyslog.d/50-default.conf

cron.*              /var/log/cron.log #将cron前面的注释符去掉 



# 重启rsyslog

$ sudo  service rsyslog  restart

查看crontab日志

$ less  /var/log/cron.log 

```

