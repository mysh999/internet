参考：

https://linuxhandbook.com/load-balancing-setup/





[TOC]



## 一、功能说明

- nginx作为web服使用
- haproxy作为HA代理
- keepalived作为haproxy单点故障检测，在第4和7层发挥作用

操作系统使用Centos7



## 二、列表

| 功能                              | IP              | 备注                 |
| --------------------------------- | --------------- | -------------------- |
| web server1(nginx)                | 192.168.159.201 |                      |
| web server2(nginx)                | 192.168.159.202 |                      |
| LoadBalancer1(haproxy+keepalived) | 192.168.159.101 | VIP：192.168.159.103 |
| LoadBalancer2(haproxy+keepalived) | 192.168.159.102 | VIP：192.168.159.103 |



## 三、配置nginx服

2台同样配置



```bash
# yum -y install epel-release
# yum -y install nginx
# systemctl start nginx
# systemctl enable nginx 
#firewall-cmd --zone=public --permanent --add-service=http
#firewall-cmd --zone=public --permanent --add-service=https
#firewall-cmd –reload 
```



第一台主机：

```shel
echo “this is first webserver” > /usr/share/nginx/html/index.html
```

第二台主机

```she
 echo "this is second webserver">/usr/share/nginx/html/index.html
```



分别访问

http://192.168.159.201/

http://192.168.159.202/



## 四、使用haproxy配置LB

2台haproxy如下配置



```shell
yum -y update
yum -y install haproxy
```



配置

```shell
#vi /etc/haproxy.cfg
global
   log /dev/log local0
   log /dev/log local1 notice
   chroot /var/lib/haproxy
   stats timeout 30s
   user haproxy
   group haproxy
   daemon
defaults
log global
mode http
option httplog
option dontlognull
timeout connect 5000
timeout client 50000
timeout server 50000

#frontend
#---------------------------------
frontend http_front
bind *:80
stats uri /haproxy?stats
default_backend http_back

#round robin balancing backend http
#-----------------------------------
backend http_back
balance roundrobin
#balance leastconn
mode http
server webserver1 192.168.159.201:80 check    # ip_address_of_1st_centos_webserver
server webserver2 192.168.159.202:80 check    # ip_address_of_2nd_centos_webserver
```



```shell
systemctl enable haproxy
systemctl start haproxy
```



查看

http://192.168.159.101/haproxy?stats

http://192.168.159.102/haproxy?stats



![1565456145425](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1565456145425.png)



## 五、使用keepalived配置haproxy的高可用

在2台haproxy服上安装keepalived

```shell
# yum -y install keepalived
```



配置

```bash
# vim /etc/keepalived/keepalived.conf 
global_defs {
notification_email {
linuxhandbook.com
linuxhandbook@gmail.com
}
notification_email_from thunderdfrost@gmail.com
smtp_server 192.168.159.1
smtp_connect_timeout 30
router_id LVS_DEVEL
}

vrrp_instance VI_1 {
state MASTER
interface ens33 #put your interface name here. [to see interface name: $ ip a ]
virtual_router_id 51
priority 101 # 101 for master. 100 for backup. [priority of master> priority of backup]
#主改成101，备改成100
advert_int 1
authentication {
auth_type PASS
auth_pass 1111 #password
}
virtual_ipaddress {
192.168.159.103 # use the virtual ip address. 
}
}
```



启动

```bash
systemctl start keepalived
systemctl enable keepalived
```



测试输出

```bash
# while true
> do
> curl 192.168.159.103
> sleep 1
> done
this is second webserver
this is first webserver
this is second webserver
this is first webserver
this is second webserver
```



## 六、模拟宕机测试

1、haproxy1宕机

表现：

服务IP飘移到haproxy2上

访问不变

```bash
[root@haproxy2 ~]# while true
> do
> curl 192.168.159.103
> sleep 1
> done
this is first webserver
this is second webserver
this is first webserver
this is second webserver
```



2、nginx1宕机

表现：

只能访问nginx2的内容

```bash
# while true; do curl 192.168.159.103; sleep 1; done
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
this is second webserver
this is second webserver
```

反之亦然