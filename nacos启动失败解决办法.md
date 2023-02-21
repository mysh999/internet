## 一、问题描述

有一台nacos服务器停电后重启，启动报以下信息：

```bash
# tail -20f nacos.log

Caused by: java.net.UnknownHostException: jmenv.tbsite.net
        at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:184)
        at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
        at java.net.Socket.connect(Socket.java:607)
        at sun.net.NetworkClient.doConnect(NetworkClient.java:175)
        at sun.net.www.http.HttpClient.openServer(HttpClient.java:463)
        at sun.net.www.http.HttpClient.openServer(HttpClient.java:558)
        at sun.net.www.http.HttpClient.<init>(HttpClient.java:242)
        at sun.net.www.http.HttpClient.New(HttpClient.java:339)
        at sun.net.www.http.HttpClient.New(HttpClient.java:357)
        at sun.net.www.protocol.http.HttpURLConnection.getNewHttpClient(HttpURLConnection.java:1228)
        at sun.net.www.protocol.http.HttpURLConnection.plainConnect0(HttpURLConnection.java:1162)
        at sun.net.www.protocol.http.HttpURLConnection.plainConnect(HttpURLConnection.java:1056)
```





## 二、原因

```bash
Nacos默认是集群（cluster）启动，将其设置为单机（standalone）启动则不会报这个错
```



## 三、启动

执行以下命令顺利启动

```bash
# ./shutdown.sh 
The nacosServer(913514) is running...
Send shutdown request to nacosServer(913514) OK



root@ubuntu20:/data/nacos/bin# ./startup.sh -m standalone
/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java -Djava.ext.dirs=/usr/lib/jvm/java-8-openjdk-amd64/jre/jre/lib/ext:/usr/lib/jvm/java-8-openjdk-amd64/jre/lib/ext  -Xms512m -Xmx512m -Xmn256m -Dnacos.standalone=true -Dnacos.member.list= -Xloggc:/data/nacos/logs/nacos_gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dloader.path=/data/nacos/plugins/health,/data/nacos/plugins/cmdb,/data/nacos/plugins/selector -Dnacos.home=/data/nacos -jar /data/nacos/target/nacos-server.jar  --spring.config.additional-location=file:/data/nacos/conf/ --logging.config=/data/nacos/conf/nacos-logback.xml --server.max-http-header-size=524288
nacos is starting with standalone
nacos is starting，you can check the /data/nacos/logs/start.out
```

