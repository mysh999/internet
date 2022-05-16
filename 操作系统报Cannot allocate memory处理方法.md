## 一、现象

阿里云主机，ssh登录报

```bash
[USM] SSH protocol handshake error, Socket error: Connection reset by peer
```

在阿里云界面上查找负载高的进程，对该进程所在的工程，禁用nginx流量指向

再登录主机，可以登录，但是执行命令还是提示

```bash
-bash: fork: Cannot allocate memory
```



## 二、解决办法

2.1、kill掉负载高的进程

2.2、修改系统参数

系统默认的pid_max 值为32768，查询现有的进程数**#cat /proc/sys/kernel/pid_max**
正常情况下是够用的，当我们跑重量任务时，会不够用，最终导致内存无法分配的错误,然而连不上的悲剧

```bash
# 查看可连接最大进程数
# cat /proc/sys/kernel/pid_max
32768

# 查询现有连接进程数
# pstree -p|wc -l
10206

# 永久修改最大进程数（重启后不生效）
#echo "kernel.pid_max=1000000 " >> /etc/sysctl.conf
#sysctl -p
```



