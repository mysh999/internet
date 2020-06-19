参考来源：

http://www.strugglesquirrel.com/2019/09/06/集群运维大宝剑之内存耗尽排查/

且经过实际案例验证



## 一、现象描述

- 系统内存被耗尽，可用内存只要1-2G

- rsyslogd进程占用内存比例高达20%



## 二、问题排查

经过排查，业务进程占用超高内存是因为发生写入阻塞，发生写入阻塞的原因是什么呢？排查监控日志判断是内存不足，导致osd缓慢，写入延迟开始增大，发生业务阻塞，然后内存耗尽，于是开启了链式反应



```bash
# systemctl status  rsyslog.service
● rsyslog.service - System Logging Service
   Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
   Active: active (running) since 二 2019-08-13 10:53:53 CST; 1 weeks 1 days ago
     Docs: man:rsyslogd(8)
           http://www.rsyslog.com/doc/
 Main PID: 5011 (rsyslogd)
   CGroup: /system.slice/rsyslog.service
           └─5011 /usr/sbin/rsyslogd -n

8月 21 08:50:55 server07 rsyslogd[5011]: action 'action 1' resumed (module 'builtin:omfile') [v8.24.0 try http://www.rsyslog.com/e/2359 ]
8月 21 08:50:55 server07 rsyslogd[5011]: action 'action 1' resumed (module 'builtin:omfile') [v8.24.0 try http://www.rsyslog.com/e/2359 ]
8月 21 08:55:01 server07 rsyslogd[5011]: action 'action 0' resumed (module 'builtin:omfile') [v8.24.0 try http://www.rsyslog.com/e/2359 ]
8月 21 08:55:01 server07 rsyslogd[5011]: action 'action 0' resumed (module 'builtin:omfile') [v8.24.0 try http://www.rsyslog.com/e/2359 ]
8月 21 23:53:02 server07 rsyslogd[5011]: action 'action 2' resumed (module 'builtin:omfile') [v8.24.0 try http://www.rsyslog.com/e/2359 ]
8月 21 23:53:02 server07 rsyslogd[5011]: action 'action 2' resumed (module 'builtin:omfile') [v8.24.0 try http://www.rsyslog.com/e/2359 ]


# /usr/bin/cat /proc/5011/status|/usr/bin/grep -w 'VmRSS'
VmRSS:     6916064 kB
```





## 三、解决办法

`限制syslog进程内存的使用，限制为1G

```bash
# systemctl edit rsyslog.service --full
#加入参数，这里我们限制rsyslogd进程使用最大物理内存为1G，因此加入LimitRSS=1073741824到service域中
[Unit]
Description=System Logging Service
;Requires=syslog.socket
Wants=network.target network-online.target
After=network.target network-online.target
Documentation=man:rsyslogd(8)
Documentation=http://www.rsyslog.com/doc/

[Service]
LimitRSS=1073741824   #添加到这里
Type=notify
EnvironmentFile=-/etc/sysconfig/rsyslog
ExecStart=/usr/sbin/rsyslogd -n $SYSLOGD_OPTIONS
Restart=on-failure
UMask=0066
StandardOutput=null
Restart=on-failure

[Install]
WantedBy=multi-user.target
;Alias=syslog.service


```



重新加载进程

```bash
# systemctl daemon-reload
# systemctl restart rsyslog.service

```



确认是否生效

```bash
# ps -ef|grep rsyslog
root     1289301       1  0 10:34 ?        00:00:00 /usr/sbin/rsyslogd -n
root     1292065 1289493  0 11:03 pts/0    00:00:00 grep --color=auto rsyslog
# cat /proc/1289301/limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          1073741824           1073741824           bytes
Max processes             255284               255284               processes
Max open files            1024                 4096                 files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       255284               255284               signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     0                    0
Max realtime timeout      unlimited            unlimited            us
```

`Max resident set`已经设置为1G，默认是unlimited的



运行一段时间后，rsyslogd进程未见内存异常占用