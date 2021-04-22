## 一、需求说明	

需要在阿里云 oss://ide-cdn/ 配置跨域设置

该bucket没有配置CDN，因此只需要在bucket上配置跨域



如果启用了CDN，按该文档在CDN中配置

https://help.aliyun.com/knowledge_detail/40183.html?spm=a2c4g.11186623.2.5.993561aaMtGwGu



## 二、配置步骤

2.1、防盗链配置

设置为允许

对象存储-权限管理-防盗链

![企业微信截图_20210422150753.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gpsjetm5i9j31b50lv0u1.jpg)





2.2、设置跨域

![企业微信截图_20210422150854.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gpsjfiovawj31c70k275c.jpg)

![企业微信截图_20210422150921.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gpsjg1wu8aj31as05tq2v.jpg)







2.3、验证

打开开发提供的代理连接

http://10.10.18.233:2222/#/

打开调试窗口，可以打开页面