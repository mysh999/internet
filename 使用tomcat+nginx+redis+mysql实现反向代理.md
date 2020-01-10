[TOC]





## 一、概念

1、目的

Tomcat+Nginx+Redis+MySQL实现反向代理、负载均衡、session共享



2、什么叫反向代理

指服务器根据客户端的请求，从其关系的一组或多组后端服务器（如Web服务器）上获取资源，然后再将这些资源返回给客户端，客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器簇的存在。



正向代理和反向区别：

正向代理的典型用途是为在防火墙内的局域网客户端提供访问Internet的途径。正向代理还可以使用缓冲特性减少网络使用率；

反向代理的典型用途是将防火墙后面的服务器提供给Internet用户访问。反向代理还可以为后端的多台服务器提供负载平衡，或为后端较慢的服务器提供缓冲服务。



## 二、环境准备



| 主机名  | IP              | 角色               | 操作系统 |
| ------- | --------------- | ------------------ | -------- |
| nginx1  | 192.168.159.101 | 反向代理、负载均衡 | Centos7  |
| tomcat1 | 192.168.159.102 | tomcat集群服       | Centos7  |
| tomcat2 | 192.168.159.103 | tomcat集群服       | Centos7  |
| redis1  | 192.168.159.104 | session共享服      | Centos7  |
| mysql1  | 192.168.159.105 | DB                 | Centos7  |

说明：以下在哪台服上操作，就以主机名说明



## 三、nginx配置

3.1 安装关联包

```bash
#yum -y install gcc-c++  pcre pcre-devel  zlib zlib-devel  openssl openssl--devel 
```



3.2  卸载原生nginx

```bash
# yum remove nginx  
# yum remove nginx-filesystem-1.12.2-2.el7.noarch
```



3.3 安装nginx

```bash
[root@nginx1 src]# tar -zxvf nginx-1.14.0.tar.gz 
[root@nginx1 src]# cd nginx-1.14.0/
[root@nginx1 nginx-1.14.0]# ./configure --with-http_stub_status_module
[root@nginx1 nginx-1.14.0]# make
[root@nginx1 nginx-1.14.0]# make install
[root@nginx1 nginx-1.14.0]# whereis nginx
nginx: /usr/lib64/nginx /usr/local/nginx /usr/share/nginx


/usr/local/nginx/sbin/nginx             //启动
/usr/local/nginx/sbin/nginx -s stop     //关闭
/usr/local/nginx/sbin/nginx -s reload   //重启

```



3.4 添加防火墙规则

```bash
[root@nginx1 nginx-1.14.0]# firewall-cmd --zone=public --add-port=80/tcp --permanent
[root@nginx1 nginx-1.14.0]# firewall-cmd --reload
```



## 四、部署tomcat

说明：2台tomcat服配置一样



4.1 卸载自带的java

```bash
[root@tomcat1 ~]# yum -y remove java*

```



4.2 安装java

解压缩jdk包
```bash
tar -zxvf jdk-8u151-linux-x64.tar.gz 
```



重命名和拷贝
```bash
# mv –f jdk1.8.0_151 jdk

# mv –f jdk /usr/local/
```



配置root用户环境变量
```bash
# vim /etc/profile 

export JAVA_HOME=/usr/local/jdk
export JRE_HOME=/usr/local/jdk/jre
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```



立即生效
```bash
source  /etc/profile
ln -s /usr/local/jdk/bin/java /usr/bin/java
```





验证java
```bash
# which java

/usr/local/jdk/bin/java

# java -version
java version "1.8.0_151"
Java(TM) SE Runtime Environment (build 1.8.0_151-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.151-b12, mixed mode)
```



4.3、安装tomcat

```bash
[root@tomcat1 src]# tar -zxvf apache-tomcat-8.0.47.tar.gz -C /usr/local/
[root@tomcat1 src]# cd /usr/local/
[root@tomcat1 local]# ln -sv apache-tomcat-8.0.47 tomcat
‘tomcat’ -> ‘apache-tomcat-8.0.47’

[root@tomcat1 local]# vim /etc/profile.d/tomcat.sh  #配置tomcat环境变量
export CATALINA_HOME=/usr/local/tomcat
export PATH=$CATALINA_HOME/bin:$PATH

[root@tomcat1 local]# source  /etc/profile.d/tomcat.sh 

[root@tomcat1 local]# catalina.sh start   #启动服务s
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk/jre
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.
```



添加防火墙规则

```bash
[root@tomcat1 local]# firewall-cmd --zone=public --add-port=8080/tcp --permanent
success
[root@tomcat1 local]# firewall-cmd --reload
success       
```





验证

```bash
[root@tomcat1 local]# netstat -tnlp|grep -i 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      2311/java   
[root@tomcat1 local]# curl http://localhost:8080
```



设置默认虚拟主机，并增加jvmRoute

```bash
# vim /usr/local/tomcat/conf/server.xml 
 <Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1">  #jvmRoute是jvm标识，就是页面最顶部的标签，在实际生产环境中，所有的后台tomcat标识都要一样，这里为了实验的说明性，两台tomcat的标识改成不一样的，分别为tomcat1和tomcat2
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Context docBase="/Data/webapps1" path="" reloadable="true" /> #修改默认虚拟主机，并将网站文件路径指向/Data/webapps1
```



创建context目录和测试页面

```bash
# mkdir -p /Data/webapps1

[root@tomcat1 local]# cat /Data/webapps1/index.jsp
<%@ page language="java" %>
<html>
  <head><title>Tomcat-1</title></head>
  <body>
    <h1><font color="red">www.tomcat-1.com</font></h1>
    <table align="centre" border="1">
      <tr>
        <td>Session ID</td>
    <% session.setAttribute("tomcat-1.com","tomcat-1.com"); %>
        <td><%= session.getId() %></td>
      </tr>
      <tr>
        <td>Created on</td>
        <td><%= session.getCreationTime() %></td>
     </tr>
    </table>
  </body>
</html>
```



测试配置，并启动

```bash
[root@tomcat1-server-1 local]# catalina.sh stop
[root@tomcat1-server-1 local]# catalina.sh configtest
[root@tomcat1-server-1 local]# catalina.sh start
```

使用类似方法配置tomcat2服



## 五、配置nginx负载均衡tomcat

```bash
[root@nginx1 ~]# cat /usr/local/nginx/conf/nginx.conf
user  nginx;
worker_processes  1;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;
    upstream tomcat-web {
      server 192.168.159.102:8080;
      server 192.168.159.103:8080;
       }
    server {
        listen       80;
        server_name  www.tomcat.com;


        location / {
            root   html;
            index  index.html index.htm index.jsp;
        }
        location ~* \.(jsp|do)$ {
        proxy_pass http://tomcat-web;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
                   }
       location /nginx_status {
        stub_status on;
        access_log off;
        allow 192.168.159.0/24;
        deny all;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```



重新载入

```bash
[root@nginx1 nginx-1.14.0]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful    

如果报
nginx: [emerg] unknown directive "stub_status" in /usr/local/nginx/conf/nginx.conf:43

原因是Nginx没有添加modules/ngx_http_stub_status_module.o模块
编译时加上
./configure  --with-http_stub_status_module

[root@nginx1 nginx-1.14.0]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf 

[root@nginx1 nginx-1.14.0]# /usr/local/nginx/sbin/nginx -s reload  

```



验证测试

http://192.168.159.101/index.jsp

或者用域名(域名添加到hosts文件中)



效果：

多刷新几次，从结果能看出，nginx把访问请求分别分发给了后端的tomcat-1和tomcat-2，客户端的访问请求实现了负载均衡，但session id不一样(即：没有实现session保持)

![1566058524299](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1566058524299.png)







![1566058546227](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1566058546227.png)



## 六、部署redis服务

编译安装

```bash
[root@redis-server ~]# tar xf redis-3.2.11.tar.gz 
[root@redis-server ~]# cd redis-3.2.11
[root@redis-server redis-3.2.3]# make
[root@redis-server redis-3.2.3]# make PREFIX=/usr/local/redis install
[root@redis1 redis-3.2.11]# make PREFIX=/usr/local/redis install
```



配置redis

```bash
[root@redis1 redis-3.2.11]# mkdir /usr/local/redis/etc
[root@redis1 redis-3.2.11]# cp redis.conf /usr/local/redis/etc/
[root@redis1 redis-3.2.11]# cd /usr/local/redis/bin/
[root@redis1 bin]# cp redis-benchmark redis-cli redis-server /usr/bin
```



调整下内存分配使用方式并使其生效

#此参数可用的值为0,1,2 
#0表示当用户空间请求更多的内存时，内核尝试估算出可用的内存 
#1表示内核允许超量使用内存直到内存用完为止 
#2表示整个内存地址空间不能超过swap+(vm.overcommit_ratio)%的RAM值 

```bash
[root@redis-server bin]# echo "vm.overcommit_memory=1">>/etc/sysctl.conf
[root@redis-server bin]# sysctl -p
```





修改redis配置

```bash
[root@redis-server bin]#vim /usr/local/redis/etc/redis.conf

# 修改一下配置

#设置redis监听的地址
bind 0.0.0.0

# redis以守护进程的方式运行

# no表示不以守护进程的方式运行(会占用一个终端)  

daemonize yes

# 客户端闲置多长时间后断开连接，默认为0关闭此功能                                      

timeout 300

# 设置redis日志级别，默认级别：notice                    

loglevel verbose

# 设置日志文件的输出方式,如果以守护进程的方式运行redis 默认:"" 

# 并且日志输出设置为stdout,那么日志信息就输出到/dev/null里面去了 

logfile "/usr/local/redis/log/redis-access.log"

#redis默认是空密码访问，这样很不安全。需要启用redis的密码验证功能
requirepass pwd@123
```



创建日志文件

```bash
[root@redis1 bin]# mkdir -p /usr/local/redis/log/
[root@redis1 bin]# touch /usr/local/redis/log/redis-access.log
```



配置redis环境变量配置

```bash
[root@redis-server bin]# echo "export PATH=/usr/local/redis/bin:$PATH" >  /etc/profile.d/redis.sh
[root@redis-server bin]# . /etc/profile.d/redis.sh
```

创建Redis 系统启动脚本

```bash
[root@redis1 bin]# cat /etc/init.d/redis                         
#!/bin/bash
  #chkconfig: 2345 80 90
  # Simple Redis init.d script conceived to work on Linux systems
  # as it does use of the /proc filesystem.

  PATH=/usr/local/bin:/sbin:/usr/bin:/bin
  REDISPORT=6379
  EXEC=/usr/local/redis/bin/redis-server
  REDIS_CLI=/usr/local/redis/bin/redis-cli
     
  PIDFILE=/var/run/redis_6379.pid
  CONF="/usr/local/redis/etc/redis.conf"
     
  case "$1" in
      start)
          if [ -f $PIDFILE ]
          then
                  echo "$PIDFILE exists, process is already running or crashed"
          else
                  echo "Starting Redis server..."
                  $EXEC $CONF
          fi
          if [ "$?"="0" ] 
          then
                echo "Redis is running..."
          fi
          ;;
      stop)
          if [ ! -f $PIDFILE ]
          then
                  echo "$PIDFILE does not exist, process is not running"
          else
                  PID=$(cat $PIDFILE)
                  echo "Stopping ..."
                  $REDIS_CLI -p $REDISPORT SHUTDOWN
                  while [ -x ${PIDFILE} ]
                 do
                      echo "Waiting for Redis to shutdown ..."
                      sleep 1
                  done
                  echo "Redis stopped"
          fi
          ;;
     restart|force-reload)
          ${0} stop
          ${0} start
          ;;
    *)
      echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2
          exit 1
  esac
```



赋予执行权限

```bash
[root@redis1 bin]# chmod +x /etc/init.d/redis   
```



启动

```bash
# service redis start
```



防火墙配置

```bash
[root@redis1 bin]#  firewall-cmd --zone=public --add-port=6379/tcp --permanent    
success
[root@redis1 bin]# firewall-cmd --reload
success
```

测试

```bash
[root@redis1 bin]# redis-cli -h 192.168.159.104 -p 6379 -a pwd@123
192.168.159.104:6379> keys *
(empty list or set)
192.168.159.104:6379> set name pwd
OK
192.168.159.104:6379> get name
"pwd"
```



## 七、配置tomcat session redis同步(tomcat服上配置)

两台都要配置



7.1 准备包

Tomcat8连接Reids需要以下3个软件包：

commons-pool2-2.2.jar
jedis-2.5.2.jar
tomcat-redis-session-manager-2.0.0.jar



7.2 将所需要的jar包复制到$CATALINA_HOME/lib/下，即tomcat安装目录的lib目录下

```bash
[root@tomcat1 src]# cp commons-pool2-2.2.jar jedis-2.5.2.jar tomcat-redis-session-manager-2.0.0.jar /usr/local/tomcat/lib/
```



7.3  在Tomcat的conf/context.xml文件中加入使用redis-session的配置

```bash
#vi context.xml
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" /> 
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager" 
host="192.168.159.104" 
password="pwd@123" 
port="6379" 
database="0" 
maxInactiveInterval="60"
 />
```



7.4 重启tomcat

```bash
[root@tomcat1 conf]# catalina.sh stop
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk/jre
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
[root@tomcat1 conf]# catalina.sh start
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk/jre
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.
```



7.5 测试

http://192.168.159.101/index.jsp



可以看出，分别访问了不同的tomcat，但是得到的session却是相同的，说明达到了集群的目的。

注：从Tomcat6开始默认开启了Session持久化设置，测试时可以关闭本地Session持久化，在Tomcat的conf目录下的context.xml文件中，取消 <Manager pathname="" /> 注释即可

![1566061918612](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1566061918612.png)





![1566061934127](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1566061934127.png)







## 八、mysql部署

8.1 安装依赖包

```bash
#yum install –y make cmake  bison gcc gcc-c++     ncurses  ncurses-devel
```



8. 2 创建mysql用户和组，默认密码mysql

```bash
#useradd -g mysql -d /home/mysql  -s /sbin/nologin --system  -M mysql
```



建立属于mysql组的mysql用户，且不允许登录

```bash
# passwd mysql

Changing password for user mysql.

New password: 

BAD PASSWORD: The password is shorter than 8 characters

Retype new password: 

passwd: all authentication tokens updated successfully.
```



新密码是abc#123

 

 8.3     安装boost

从MySQL 5.7.5开始Boost库是必需的

```bash
[root@mysql1 src]# tar -zxvf boost_1_59_0.tar.gz -C /usr/local/
[root@mysql1 src]# cd /usr/local/boost_1_59_0/
[root@mysql1 boost_1_59_0]# ./bootstrap.sh
[root@mysql1 boost_1_59_0]# ./b2 install
```



8.4 安装mysql

```bash
[root@mysql1 src]# mkdir -p /usr/local/mysql/data /usr/local/mysql/binlog
[root@mysql1 src]# tar -zxvf mysql-5.7.18.tar.gz -C /usr/local/
[root@mysql1 src]# cd /usr/local/mysql-5.7.18/
[root@mysql1 mysql-5.7.18]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql  -DMYSQL_DATADIR=/usr/local/mysql/data  -DDOWNLOAD_BOOST=1  -DWITH_BOOST=/usr/local/boost_1_59_0  -SYSCONFDIR=/etc  -DWITH_MYISAM_STORAGE_ENGINE=1  -DWITH_INNOBASE_STORAGE_ENGINE=1  -DWITH_MEMORY_STORAGE_ENGINE=1  -DWITH_READLINE=1  -DMYSQL_UNIX_ADDR=/tmp/mysqld.sock  -DMYSQL_TCP_PORT=3306  -DENABLED_LOCAL_INFILE=1  -DWITH_PARTITION_STORAGE_ENGINE=1  -DEXTRA_CHARSET=all  -DDEFAULT_CHARSET=utf8  -DDEFAULT_COLLATION=utf8_general_ci
[root@mysql1 mysql-5.7.18]# make
[root@mysql1 mysql-5.7.18]# make install
[root@mysql1 mysql-5.7.18]# touch /usr/local/mysql/mysql_error.log       
[root@mysql1 mysql-5.7.18]# chown -R mysql:mysql /usr/local/mysql
[root@mysql1 mysql-5.7.18]# chmod u+w /usr/local/mysql/

#配置开机自启动
[root@mysql1 mysql-5.7.18]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
[root@mysql1 mysql-5.7.18]# chmod +x /etc/init.d/mysqld 
[root@mysql1 mysql-5.7.18]# chkconfig --add mysqld
[root@mysql1 mysql-5.7.18]# chkconfig mysqld on


#配置my.cnf
[root@mysql1 mysql-5.7.18]# cat /etc/my.cnf
[client]
port=3306
socket=/usr/local/mysql/data/mysql.sock
default-character-set =utf8
[mysqld]
port=3306
user=mysql
server-id = 111
log_bin=/usr/local/mysql/binlog/mysql-bin     #最后面是日志名前缀
expire_logs_days = 7
socket=/usr/local/mysql/data/mysql.sock
pid-file=/usr/local/mysql/data/mysql.pid
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
tmpdir=/tmp
default-time-zone= system
character-set-server =utf8
default-storage-engine =InnoDB
log_error=/usr/local/mysql/mysql_error.log

#初始化mysql权限表
# cd /usr/local/mysql/bin
# ./mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

#启动数据库
#/etc/init.d/mysqld start
# ./mysql -u root -p
Enter password:  //回车

 第一次登录提示要修改密码
 mysql> SET PASSWORD = PASSWORD('root');
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> FLUSH PRIVILEGES; 
Query OK, 0 rows affected (0.00 sec)

再次登录，输入新密码
# ./mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.18 Source distribution

#配置环境变量 
[root@mysql1 bin]# vim /etc/profile
export PATH=/usr/local/mysql/bin:$PATH

[root@mysql1 bin]# source  /etc/profile

#配置防火墙
[root@mysql1 bin]#  firewall-cmd --zone=public --add-port=3306/tcp --permanent    
success
[root@mysql1 bin]# firewall-cmd --reload
success

#创建连接库
mysql> create database javatest;

#创建认证用户
mysql> grant all on *.* to tomcat_user@'192.168.159.%' identified by '123456';
mysql> flush privileges;

```





## 九、配置tomcat连接数据库（tomcat服操作）

以下2台tomcat服操作

9.1 下载mysql-connector-java-5.1.22-bin.jar并复制到$CATALINA_HOME/lib目录下

```bash
[root@tomcat1 lib]# unzip mysql-connector-java-5.1.22-bin.jar.zip 
```



9.2 配置JNDI数据源

```bash
[root@tomcat1 conf]# cat /usr/local/tomcat/conf/context.xml    
<?xml version='1.0' encoding='utf-8'?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<!-- The contents of this file will be loaded for each web application -->
<Context>
       ###配置DB源
       <Resource name="jdbc/TestDB" auth="Container"
        type = "javax.sql.DataSource" maxActive = "100" maxIdle="30"
        maxWait="10000" username="tomcat_user" password="123456" 
        driverClassName = "com.mysql.jdbc.Driver"
        url="jdbc:mysql://192.168.159.105:3306/javatest"/>

    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->

    <!-- Uncomment this to enable Comet connection tacking (provides events
         on session expiration as well as webapp lifecycle) -->
    <!--
    <Valve className="org.apache.catalina.valves.CometConnectionManagerValve" />
    -->
###配置redis
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" /> 
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager" 
host="192.168.159.104" 
password="pwd@123" 
port="6379" 
database="0" 
maxInactiveInterval="60"
 />
</Context>
```



9.3 在项目的目录下新建WEB-INF目录，用于存放网站xml配置文件，用于tomcat连接mysql数据库

```bash
[root@tomcat1 conf]# mkdir /Data/webapps1/WEB-INF
[root@tomcat1 WEB-INF]# cat /Data/webapps1/WEB-INF/web.xml 
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
          http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
           version="3.0">
    <resource-ref>
        <res-ref-name>jdbc/demo</res-ref-name>
        <res-type>javax.sql.DataSource</res-type>
        <res-auth>Container</res-auth>
        <res-sharing-scope>Shareable</res-sharing-scope>
    </resource-ref>
</web-app>
```



9.4 重启服务

```bash
[root@tomcat1 WEB-INF]#  catalina.sh stop
[root@tomcat1 WEB-INF]#  catalina.sh start
```



9.5 创建测试页面，测试tomcat和mysql连通性

```bash
[root@tomcat1 webapps1]# cat /Data/webapps1/test.jsp 
<%@page import="javax.naming.InitialContext"%>
<%@page import="javax.sql.DataSource"%>
<%@page import="java.sql.Connection"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"  
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    <%
        out.print("MySQL 数据源测试开始..." + "<br/>");
        DataSource ds = null;
        try {
            InitialContext ctx = new InitialContext();
            ds = (DataSource) ctx.lookup("java:comp/env/jdbc/TestDB");
            Connection conn = ds.getConnection();
            conn.close();
            out.print("MySQL 数据源测试成功！");
        } catch (Exception ex) {
            out.print("出现意外，信息是:" + ex.getMessage());
            ex.printStackTrace();
        }
    %>
    %    %</body>
    %        %</html>
```



9.6 连通检测

http://192.168.159.102:8080/test.jsp

![1566111223920](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1566111223920.png)



附tomcat2的配置：

```bash
[root@tomcat2 conf]# cat /usr/local/tomcat/conf/context.xml 
<?xml version='1.0' encoding='utf-8'?>
<!--
       Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<!-- The contents of this file will be loaded for each web application -->
<Context>
       
       <Resource name="jdbc/TestDB" auth="Container"
        type = "javax.sql.DataSource" maxActive = "100" maxIdle="30"
        maxWait="10000" username="tomcat_user" password="123456" 
        driverClassName = "com.mysql.jdbc.Driver"
        url="jdbc:mysql://192.168.159.105:3306/javatest"/>

    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
             <Manager pathname="" />
    -->

    <!-- Uncomment this to enable Comet connection tacking (provides events
                  on session expiration as well as webapp lifecycle) -->
    <!--
             <Valve className="org.apache.catalina.valves.CometConnectionManagerValve" />
    -->

<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" /> 
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager" 
host="192.168.159.104" 
password="pwd@123" 
port="6379" 
database="0" 
maxInactiveInterval="60"
 />
</Context>






```

