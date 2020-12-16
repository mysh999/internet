在Ubuntu18.04版本测试通过

ubuntu18.04不能像其他版本ubuntu一样直接编辑rc.local文件来设置开机自启动脚本，通过以下方法配置

 

## 一、建立rc-local.service文件

```bash
$ sudo vi /etc/systemd/system/rc-local.service
[Unit]
Description=/etc/rc.local Compatibility
Documentation=man:systemd-rc-local-generator(8)
ConditionFileIsExecutable=/etc/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no

# 追加的内容
[Install]
WantedBy=multi-user.target
Alias=rc-local.service
```



## 二、创建rc.local文件

```bash
# 没有创建它
$sudo touch /etc/rc.local 

# 添加启动内容
$ cat /etc/rc.local 
#!/bin/sh -e
sh /data/ubt/start_svc.sh
exit 0

# 给文件添加执行权限
$ sudo chmod +x /etc/rc.local
```



## 三、开启服务

```bash
$ sudo systemctl start rc-local.service

$ sudo systemctl status rc-local.service
$ sudo systemctl enable rc-local
```

