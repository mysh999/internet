由于rabbitmq3.0以后的版本默认guest只能从localhost连接不能使用远程连接，所以我们要设置外网连接

找到rabbitmq的安装位置，cd进去然后在里面找到etc/rabbitmq，进入后，在etc/rabbitmq/下面新建文件：

```bash
#touch rabbitmq.config
```

然后将

```bash
[{rabbit, [{loopback_users, []}]}].
```



放到里面

cd rabbitmq/sbin 重新进入你rabbitmq安装目录，然后进入sbin文件中，执行命令重新启动rabbitmq即可：

```bash
# ./rabbitmqctl stop
# ./rabbitmq-server
```







