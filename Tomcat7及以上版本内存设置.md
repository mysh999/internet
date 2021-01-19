在tomcat_home/bin目录下找到catalina.sh，用文本编辑器打开，加上下面一行

```bash
set JAVA_OPTS= -Xms1024M -Xmx1024M -XX:PermSize=256M -XX:MaxNewSize=256M -XX:MaxPermSize=256M
解释一下各个参数：

-Xms1024M：初始化堆内存大小（注意，不加M的话单位是KB）

-Xmx1024M：最大堆内存大小

-XX:PermSize=256M：初始化类加载内存池大小

-XX:MaxPermSize=256M：最大类加载内存池大小

-XX:MaxNewSize=256M：这个还不清楚哈，有知道的说声

还有一个-server参数，是指启动jvm时以服务器方式启动，比客户端启动慢，但性能较好，大家可以自己选择。
```

