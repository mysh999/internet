





[TOC]



## 一、简介

​			pinpoint是开源的一款APM监控工具，用Java编写，用于大规模分布式系统监控。它对性能的影响最小，安装agent是无侵入式的。



​		pinpoint提供了一些功能：

- ​		服务映射：通过可视化其组件如何互连来了解任何分布式系统的关联关系。单击节点可显示有关组件的详细信息，例如其当前状态和事务计数。

- ​		实时的活跃线程数

- ​		请求/响应散点图

- ​		调用栈

- ​		查看有关应用程序的其他详细信息，例如CPU使用率，内存/垃圾收集，TPS和JVM参数

示意图：

![20200302_1641_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gcfob8703sj30is0akgoh.jpg)



整个pinpoint架构分为3部分：pinpoint-collector、pinpoint-agent、pinpoint-webUI。

- pinpoint-agent：用来收集单个应用的信息，并将收集好的应用信息发送到pinpoint-collector中
- pinpoint-collector：用来处理pinpoint-agent发送过来的信息，并将信息收集好之后存储到HBase中
- pinpoint-webUI：查找出HBase中的数据并展示



## 二、环境准备

| IP              | 主机名      | 作用                                                         |
| --------------- | ----------- | ------------------------------------------------------------ |
| 192.168.230.200 | server1     | 安装pinpoint-collector、pinpoint-webUI、HBase                |
| 192.168.230.201 | agent1      | 安装pinpoint-agent，负责收集应用的信息、同时在上面部署tomcat、spring boot工程来做监控 |
| 192.168.230.202 | mysqlserver | 存放mysql数据、同时部署redis、zk（spring boot工程指向）      |



## 三、server1配置

### 3.1、安装java

```bash
#  卸载自带java
# yum -y remove java*

# rpm -ivh jdk-8u171-linux-x64.rpm 

#vi /etc/profile
export JAVA_HOME=/usr/bin/java
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

```



### 3.2、安装hbase

3.2.1、解压缩

```bash
# tar -zxvf hbase-1.2.12-bin.tar.gz 
```



3.2.2、配置hbase

```bash
# vi ./hbase-1.2.12/conf/hbase-env.sh 
export JAVA_HOME=/usr/java/jdk1.8.0_171-amd64

# vi ./hbase-1.2.12/conf/hbase-site.xml 
<configuration>
            <property>
                <name>hbase.rootdir</name>
                <value>file:///app/data/hbase</value>
            </property>
            <property>
                <name>hbase.zookeeper.property.dataDir</name>
                <value>/app/data/zookeeper</value>
            </property>
                <property>
                 <name>hbase.zookeeper.property.clientPort</name>
                 <value>2181</value>
                 <description>Property from ZooKeeper'sconfig zoo.cfg. The port at which the clients will connect.
                 </description>
                </property>
                <!-- false是单机模式，true是分布式模式  -->
                <property>
                 <name>hbase.cluster.distributed</name>
                 <value>false</value>
                </property>
</configuration>

#创建目录
# mkdir -p /app/data/hbase
# mkdir -p /app/data/zookeeper

```



3.2.3、启动

```bash
# ./hbase-1.2.12/bin/start-hbase.sh 
starting master, logging to /opt/hbase-1.2.12/bin/../logs/hbase-root-master-server1.out
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0


# 查看Hbase是否启动成功，如果启动成功的会看到"HMaster"的进程
# jps
2898 Jps
2245 HMaster
```



3.2.4、初始化pinpoint库

​           下载库文件

```bas
# wget https://raw.githubusercontent.com/naver/pinpoint/master/hbase/scripts/hbase-create.hbase
```



​			执行初始化

```bash
# ./hbase-1.2.12/bin/hbase shell /opt/hbase-create.hbase 

# 查看初始化的表是否存在
# ./hbase-1.2.12/bin/hbase shell 
2020-03-02 18:02:39,461 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.12, r91d5ec4c4dcd10ceec984c6e663ea82acf353995, Sat Apr  6 15:27:28 CDT 2019

hbase(main):001:0> status 'detailed'
```

也可以登录查看初始化数据

http://192.168.230.200:16010/master-status

![20200302_1805_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gcfqqzliqbj31170ian9z.jpg)



### 3.3、安装pinpoint-collector

3.3.1、端口说明

| 端口  | 描述                                                         |
| :---- | :----------------------------------------------------------- |
| 28005 | Tomcat 关闭端口(默认 8005)                                   |
| 28080 | Tomcat HTTP端口(默认 8080)                                   |
| 28443 | Tomcat HTTPS端口(默认 8443)                                  |
| 28009 | Tomcat AJP端口(默认 8009)                                    |
| 9994  | collector与agent通信 TCP端口(默认 9994),`collector.tcpListenPort`与agent配置文件中`profiler.collector.tcp.port`一致 |
| 9995  | collector与agent通信 UDP端口(默认 9995),`collector.udpStatListenPort`与agent配置文件中`profiler.collector.stat.port`一致 |
| 9996  | collector与agent通信 UDP端口(默认 9996),`collector.udpSpanListenPort`与agent配置文件中`profiler.collector.span.port`一致 |



3.3.2、准备tomcat

```bash
#  tar -zxvf apache-tomcat-8.0.47.tar.gz -C /usr/local
#  mv apache-tomcat-8.0.47 tomcat_collector

# 修改server.xml配置文件
# sed -i 's/8005/28005/g' /usr/local/tomcat_collector/conf/server.xml
# sed -i 's/8080/28080/g' /usr/local/tomcat_collector/conf/server.xml
# sed -i 's/8443/28443/g' /usr/local/tomcat_collector/conf/server.xml
# sed -i 's/8009/28009/g' /usr/local/tomcat_collector/conf/server.xml
# sed -i "s/localhost/`ifconfig eth0|grep 'inet '|awk '{print $2}'`/g" /usr/local/tomcat_collector/conf

# 删除Tomcat自带web文件
# rm -rf /usr/local/tomcat_collector/webapps/*

# 部署collector代码
# unzip pinpoint-collector-1.8.5.war -d /usr/local/tomcat_collector/webapps/ROOT

# 修改配置,Hbase注册的ZK信息（可选，如没有独立的zk可不做）
# sed -i '/hbase.client.host/ s/localhost/devopt.ubtrobot.com/g' /usr/local/tomcat_collector/webapps/ROOT/WEB-INF/classes/hbase.properties

# 启动
# cd /usr/local/tomcat_collector/bin/; ./startup.sh

# 查看日志，是否成功启动
# tail -f ../logs/catalina.out
```





## 四、部署数据库

4.1、安装数据库

​		过程略



4.2、创建数据库

```sql
mysql> CREATE DATABASE `pinpoint` ;

mysql> grant all on pinpoint.* to pinpoint@'%' identified by 'pinpoint';

mysql> flush privileges;
```



4.3、初始化数据库

```bash
# wget https://raw.githubusercontent.com/naver/pinpoint/master/web/src/main/resources/sql/CreateTableStatement-mysql.sql
# wget https://raw.githubusercontent.com/naver/pinpoint/master/web/src/main/resources/sql/SpringBatchJobRepositorySchema-mysql.sql

mysql> source /opt/CreateTableStatement-mysql.sql;
mysql> source /opt/SpringBatchJobRepositorySchema-mysql.sql;

```





## 五、部署Pinpoint web

5.1、准备tomcat

```bash
#  tar -zxvf apache-tomcat-8.0.47.tar.gz -C /usr/local
#  mv apache-tomcat-8.0.47 tomcat_ppweb


修改server.xml配置文件

sed -i 's/8005/28006/g' /usr/local/tomcat_ppweb/conf/server.xml
sed -i 's/8080/28081/g' /usr/local/tomcat_ppweb/conf/server.xml
sed -i 's/8443/28444/g' /usr/local/tomcat_ppweb/conf/server.xml
sed -i 's/8009/28010/g' /usr/local/tomcat_ppweb/conf/server.xml
sed -i "s/localhost/`ifconfig eth0|grep 'inet '|awk '{print $2}'`/g" /usr/local/tomcat_ppweb/conf/server.xml



# 删除Tomcat自带web文件
# rm -rf /usr/local/tomcat_ppweb/webapps/*

# 部署web代码
# unzip pinpoint-web-1.8.5.war  -d /usr/local/tomcat_ppweb/webapps/ROOT

# 修改配置,Hbase注册的ZK信息（可选，如没有独立的zk可不做）
# sed -i '/hbase.client.host/ s/localhost/devopt.ubtrobot.com/g' /usr/local/tomcat_ppweb/webapps/ROOT/WEB-INF/classes/hbase.properties


- 修改数据库配置

# /usr/local/tomcat_ppweb/webapps/ROOT/WEB-INF/classes/jdbc.properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://192.168.230.202:3306/pinpoint?characterEncoding=UTF-8
jdbc.username=pinpoint
jdbc.password=pinpoint


# 启动
# cd /usr/local/tomcat_ppweb/bin/; ./startup.sh

# 查看日志，是否成功启动
# tail -f ../logs/catalina.out
```



浏览器中打开

http://192.168.230.200:28081/#/main

![20200302_1943_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gcftkmi1kkj30zr0ejtc7.jpg)









## 六、部署Pinpoint-agent

6.1、解压agent

```bash
# tar -zxvf pinpoint-agent-1.8.5.tar.gz 

# 修改collect ip
# vim pinpoint.config 
profiler.collector.ip=192.168.230.200

#新增，对spring boot工程的监控，不然不显示mysql等图标
profiler.applicationservertype=SPRING_BOOT

#改成false
profiler.tomcat.conditional.transform=false
```





6.2、部署测试tomcat

```bash
#  tar -zxvf apache-tomcat-8.0.47.tar.gz -C /usr/local

# vi /usr/local/apache-tomcat-8.0.47/bin/catalina.sh 
添加以下内容

CATALINA_OPTS="$CATALINA_OPTS -javaagent:/opt/pinpoint-agent/pinpoint-bootstrap-1.8.5.jar"  #agent jar包位置
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=pp20200304"  #agent id，唯一自定义
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=testTomcat"   #项目名称，自定义
```



6.3、启动tomcat

```bash
# ./startup.sh 
Using CATALINA_BASE:   /usr/local/apache-tomcat-8.0.47
Using CATALINA_HOME:   /usr/local/apache-tomcat-8.0.47
Using CATALINA_TMPDIR: /usr/local/apache-tomcat-8.0.47/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /usr/local/apache-tomcat-8.0.47/bin/bootstrap.jar:/usr/local/apache-tomcat-8.0.47/bin/tomcat-juli.jar
Tomcat started.
```



6.4、监控tomcat

访问一次tomcat页面

http://192.168.230.201:8080/





打开pp-web

[http://192.168.230.200:28081](http://192.168.230.200:28081/)，多了一个app

![20200304_1759_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gci1t2qrh4j30yf0dm42c.jpg)







successful次数多一次

![20200304_1801_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gci1vwxi3bj311k0j9453.jpg)





## 七、监控spring boot工程

7.1、说明

这里以abis工程为例，对该工程做监控



7.2、在mysql服上部署abis工程需要用到的mysql库、redis库、zk

- 导入测试服导出的mysql数据，导入到mysql服上

  过程略

  创建用户

  ```bash
  mysql> grant all on abiclouddb.* to ubx@'%' identified by 'ubx';
  ```

  

- 部署redis环境

  使用docker部署

  ```bash
  # cat docker-compose.yml 
  redis:
    image: redis:4.0.11
    container_name: my_redis222
    environment:
      TZ: Asia/Shanghai
    command: redis-server /usr/local/etc/redis/redis.conf
    ports:
      - "6379:6379"
    volumes:
      - ./data:/data
      - ./config/redis.conf:/usr/local/etc/redis/redis.conf
      - ./logs:/redis/log
    restart: always
  ```

  

- 部署zk环境

  ```bash
  # cat docker-compose.yml 
  version: '2'
  services:
      zoo1:
          image: zookeeper:3.5.5
          restart: always
          container_name: zoo1
          ports:
              - "2181:2181"
          volumes:
          - "./zoo1/data:/data"
          - "./zoo1/datalog:/datalog"
          environment:
              ZOO_MY_ID: 1
              ZOO_SERVERS: server.1=zoo1:2888:3888 
  ```

- 启动以上服务

  

7.3、从开发服拷贝cbis工程库到agent服，修改以下内容

```bash
# egrep -v "#|^$" ./conf/jedis.properties 

jedis.pool.configstr=jedis-center1:192.168.230.202:6379   #redis库信息

jedis.pool.password=

jedis.pool.timeout=3000

jedis.pool.max-total=300

jedis.pool.max-idle=200

jedis.pool.max-wait-millis=1000
```



```bash
# cat ./conf/application.properties |egrep -v "#|^$"                
server.port=8050
server.context-path=/abis-web
server.max-http-header-size=4048576

mybatis.mapperLocations=classpath*:com/ubtechinc/**/*.xml
spring.aop.proxy-target-class=true
mybatis.configuration.log-impl= org.apache.ibatis.logging.stdout.StdOutImpl

spring.swagger.open=true

```



```bash
# cat ./conf/dubbo.properties |egrep -v "#|^$"
dubbo.application-name=cbis-api-dubbo

dubbo.scan-package= com.ubtechinc.service

dubbo.protocol-name=dubbo


dubbo.protocol-accessLog=true

dubbo.protocol-port=20882

dubbo.provider-timeout=3000

dubbo.provider-retries=1

dubbo.provider-delay=-1

dubbo.registry-protocol=zookeeper

dubbo.registry-address=zookeeper://10.10.20.30:2181
```



```bash
# vi  ./conf/wrapper.conf
wrapper.java.additional.1=-DFOREX_API_HOME=.
wrapper.java.additional.2=-server
wrapper.java.additional.3=-XX:MaxPermSize=64M
wrapper.java.additional.4=-XX:+AggressiveOpts
wrapper.java.additional.5=-XX:MaxDirectMemorySize=2G
wrapper.java.additional.6=-Dfile.encoding=UTF-8
wrapper.java.additional.7=-Xmx4G
wrapper.java.additional.8=-Xms1G

#以下是新增的在spring boot中加入pinpoint-agent配置
#agent jar包所在路径
wrapper.java.additional.9=-javaagent:/opt/pinpoint-agent/pinpoint-bootstrap-1.8.5.jar  

#agent id
wrapper.java.additional.10=-Dpinpoint.agentId=20180508

#项目名称
wrapper.java.additional.11=-Dpinpoint.applicationName=forex-api
```



- 启动spring boot工程

- 打开工程接口，模拟会话生成

  http://192.168.230.201:8050/abis-web/swagger-ui.html#

  ![微信截图_20200305172842.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gcj6r63q0oj31da0iu3zv.jpg)

![微信截图_20200305173445.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gcj6rdm72bj315s0nidgz.jpg)



7.4、在pp-web上查看

![微信截图_20200305173753.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gcj6t958gpj30zg0k03zs.jpg)