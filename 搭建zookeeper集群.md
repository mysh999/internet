参考来源:

https://juejin.im/post/5ba879ce6fb9a05d16588802





## 一、什么是zookeeper

Zookeeper 分布式服务框架是 Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等等





## 二、概念



2.1、角色分类

![360截图1847012590122136.png](http://ww1.sinaimg.cn/large/007Xg1efgy1g9swb4cnxej30g807a0v4.jpg)



2.2、网络架构

​         Zookeeper的工作集群可以简单分成两类，一个是Leader，唯一一个，其余的都是follower，如何确定Leader是通过内部选举确定的

- ​		  Leader和各个follower是互相通信的，对于zk系统的数据都是保存在内存里面的，同样也会备份一份在磁盘上

- ​          对于每个zk节点而言，可以看做每个zk节点的命名空间是一样的，也就是有同样的数据。


- ​          如果Leader挂了，zk集群会重新选举，在毫秒级别就会重新选举出一个Leaer。


- ​          集群中除非有一半以上的zk节点挂了，zk service才不可用。正是基于这个特性，要将 ZK 集群的节点数量要为奇数（2n+1：如 3、5、7 个节点）较为合适。



## 三、配置过程

以下未加以说明，均是各节点都要配置

3.1、 配置hosts文件

```bash
#vi /etc/hosts
192.168.255.100 zk1
192.168.255.101 zk2
192.168.255.102 zk3
```



3.2、安装java

```bash
# tar -zxvf jdk-8u151-linux-x64.tar.gz  -C /usr/local/
#vi .bash_profile
PATH=$PATH:$HOME/bin:/usr/local/jdk1.8.0_151/bin

export PATH

#source .bash_profile
```



3.3 、安装zookeeper

```bash
# wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
# tar -zxvf zookeeper-3.4.14.tar.gz -C /usr/local/

```



3.4、在各服务器节点data 目录下（/data/zookeeper/data）创建名为 myid 的文件，在文件第一行写上对应的 Server id。这个id必须在集群环境中服务器标识中是唯一的，且大小在1～255之间

```bash
server-01:
#echo "1" > /data/zookeeper/data/myid

server-02:
#echo "2" > /data/zookeeper/data/myid

server-03:
#echo "3" > /data/zookeeper/data/myid
```



3.5 、修改配置文件

将 ./conf 目录下的 zoo_sample.cfg 文件拷贝一份，命名为 zoo.cfg

```bash
#vi zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
clientPort=2181
autopurge.snapRetainCount=10
autopurge.purgeInterval=1
server.1=zk1:2888:3888
server.2=zk2:2888:3888
server.3=zk2:2888:3888


#创建data和log目录
#mkdir -p /data/zookeeper/data
#mkdir -p /data/zookeeper/logs
```

参数说明：

- tickTime=2000
  Zookeeper最小时间单元，单位为ms，默认值为3000。也就是Leader与Follower每隔tickTime时间就会发送一个心跳。

- initLimit=10
  Leader服务器等待Follower启动并完成数据同步的时间，默认值10。
  当已经超过10*tickTime后，Leader还没有收到Follower的返回信息,那么表明这个Follower连接或同步失败。

- syncLimit=5
  Leader服务器和Follower之间进行心跳检测的最大延时时间，默认值5，最长不能超过5*tickTime

- dataDir=/home/{$user}/zookeeper/zookeeper-3.4.12/data
  存放内存数据结构的snapshot，便于快速恢复，默认情况下，事务日志也会存储在这里。建议同时配置参数dataLogDir, 事务日志的写性能直接影响zk性能。

- dataLogDir=/home/{$user}/zookeeper/zookeeper-3.4.12/data
  dataLogDir事务日志输出目录。为了达到性能最大化，一般建议把dataDir和dataLogDir分到不同的磁盘上

- autopurge.purgeInterval, autopurge.snapRetainCount
  客户端在与zookeeper交互过程中会产生非常多的日志，而且zookeeper也会将内存中的数据作为snapshot保存下来，这些数据是不会被自动删除的，这样磁盘中这样的数据就会越来越多。不过可以通过这两个参数来设置，让zookeeper自动删除数据。
  autopurge.purgeInterval：指定自动清理快照文件和事务日志文件的时间，单位为h，默认为0表示不自动清理，这个时候可以使用脚本zkCleanup.sh手动清理。如果不清理则磁盘空间占用越来越大。
  autopurge.snapRetainCount：用于指定保留快照文件和事务日志文件的个数，默认为3。
  不过如果你的集群是一个非常繁忙的集群，然后又碰上这个删除操作，可能会影响zookeeper集群的性能，所以一般会让这个过程在访问低谷的时候进行，但是遗憾的是zookeeper并没有设置在哪个时间点运行的设置，所以有的时候我们会禁用这个自动删除的功能，而在服务器上配置一个cron，然后在凌晨来干这件事。

- clientPort=2181
  客户端连接zookeeper服务的端口。这是一个TCP port。

- server.id=IP/Host : port1 : port2
  id：用来配置ZK集群中的各节点，并建议id的值和myid保持一致。
  IP/Host: 服务器的 IP 或者是与 IP 地址做了映射的主机名
  port1：Leader和Follower或Observer交换数据使用
  port2：用于Leader选举
  注意：如果是伪集群的配置方式，不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。

- maxClientCnxns
  对于一个客户端的连接数限制，默认是60

- minSessionTimeout, maxSessionTimeout
  客户端连接zookeeper的时候，都会设置一个session timeout，如果超过这个时间client没有与zookeeper server有联系，则这个session会被设置为过期





3.6、添加防火墙规则

```bash
#firewall-cmd --zone=public --add-port=2181/tcp --permanent  
#firewall-cmd --zone=public --add-port=2888/tcp --permanent 
#firewall-cmd --zone=public --add-port=3888/tcp --permanent 

#重新载入
#firewall-cmd --reload
```



3.7、启动

```bash
#以后台执行
# /usr/local/zookeeper-3.4.14/bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.14/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

#如果要前台执行查看日志
# /usr/local/zookeeper-3.4.14/bin/zkServer.sh start -foreground

#查看状态
# /usr/local/zookeeper-3.4.14/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.14/bin/../conf/zoo.cfg
Mode: leader   #其中1个节点是leader，其他节点是follower

#重启
# /usr/local/zookeeper-3.4.14/bin/zkServer.sh restart
```

