## 一、集群说明

采用容器部署方式，部署2节点的emqx集群（节点数量应该是奇数，没有多余的服务器只能配置2个）

服务器没有占用以下端口：

```
8883
1883
11883
18083
8081
8083
8084
```



## 二、容器部署

- node109配置文件

```yaml
# cat docker-compose.yml 
version: '3'
services:
 emqx:
   image: emqx/emqx:v4.1.1
   container_name: emqx
   restart: always
   environment:
     - EMQX_LISTENER__TCP__EXTERNAL=1883
     - EMQX_LOADED_PLUGINS=emqx_management,emqx_auth_redis,emqx_recon,emqx_retainer,emqx_dashboard
     - EMQX_AUTH__MYSQL__SERVER=172.31.0.50:3306
     - EMQX_AUTH__MYSQL__USERNAME=*****    #用户名
     - EMQX_AUTH__MYSQL__PASSWORD=*****    #密码
     - EMQX_AUTH__MYSQL__DATABASE=mqtt
     - EMQX_AUTH__MYSQL__PASSWORD_HASH=sha256
     - EMQX_ALLOW_ANONYMOUS=false
     - EMQX_ACL_NOMAtCH=deny
     - EMQX_MQTT__MAX_PACKET_SIZE=10MB
     - EMQX_NAME=emqx
     - EMQX_HOST=pro109.emqx.io
     - EMQX_CLUSTER__DISCOVERY=static
     - EMQX_CLUSTER__STATIC__SEEDS=emqx@pro109.emqx.io,emqx@pro32.emqx.io
   extra_hosts:
     - "pro109.emqx.io:172.31.0.49"
     - "pro32.emqx.io:172.31.0.56"
   network_mode: host
```



- node32配置文件

  ```yaml
  # cat docker-compose.yml 
  version: '3'
  services:
   emqx:
     image: emqx/emqx:v4.1.1
     container_name: emqx
     restart: always
     environment:
       - EMQX_LISTENER__TCP__EXTERNAL=1883
       - EMQX_LOADED_PLUGINS=emqx_management,emqx_auth_redis,emqx_recon,emqx_retainer,emqx_dashboard
       - EMQX_AUTH__MYSQL__SERVER=172.31.0.50:3306
       - EMQX_AUTH__MYSQL__USERNAME=*****      #用户名
       - EMQX_AUTH__MYSQL__PASSWORD=*****      #密码
       - EMQX_AUTH__MYSQL__DATABASE=mqtt
       - EMQX_AUTH__MYSQL__PASSWORD_HASH=sha256
       - EMQX_ALLOW_ANONYMOUS=false
       - EMQX_ACL_NOMAtCH=deny
       - EMQX_MQTT__MAX_PACKET_SIZE=10MB
       - EMQX_NAME=emqx
       - EMQX_HOST=pro32.emqx.io
       - EMQX_CLUSTER__DISCOVERY=static
       - EMQX_CLUSTER__STATIC__SEEDS=emqx@pro109.emqx.io,emqx@pro32.emqx.io
     extra_hosts:
       - "pro109.emqx.io:172.31.0.49"
       - "pro32.emqx.io:172.31.0.56"
     network_mode: host
  ```

  

## 三、功能验证

3.1、登陆emqx界面

输入

```html
http://任意节点IP:18083/
```



输入默认用户名admin/123456



3.2、查看节点信息

![企业微信截图_20200805114741.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfshzf67xj30m10b9aby.jpg)



3.3、启用mysql插件

该功能默认没启用，可能是bug，要手动启动（每个节点都要启用）

![企业微信截图_20200805103729.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfqjryzwvj31fm0q3mzv.jpg)





## 四、客户端验证

4.1、安装MQTTBox客户端



4.2、创建MQTTK客户端



![360截图17420917579380.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfqqqfv5pj30ql09wdg7.jpg)

4.3、配置客户端

![企业微信截图_20200805112749.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfrxihq6mj31e40fkdgo.jpg)

点击save后显示已连接

![企业微信截图_20200805110251.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfr7f3qbuj30zs0k7aao.jpg)



按照同样方法，创建另一个节点的客户端







4.4、测试消息订阅

配置第1个节点，点击订阅

![360截图18000909616187.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfs52ycofj30yt0jzgn3.jpg)



同理，配置第2个节点，订阅

![企业微信截图_20200805113710.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfs7593fbj30yj0k0mxz.jpg)





4.5、点击发布

点击第1个节点的发布，2个节点都能收到发布的信息

![企业微信截图_20200805113926.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfs9mzzkgj30ym0k7wfh.jpg)

第2个节点也能收到消息

![企业微信截图_20200805114014.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfsajeim5j30yq0jk3zg.jpg)





同理，在第2个节点发布消息，2个节点也都能收到信息



4.6、界面上查看订阅

![企业微信截图_20200805114629.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghfsgsf5nsj312z0mmjtq.jpg)