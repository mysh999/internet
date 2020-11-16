## 一、需求说明

由于nginx使用的ssl证书快要到期，需要替换新的证书

新的证书已经购买，如下

![企业微信截图_20201116135431.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gkqyzv1x6oj306g02k0sk.jpg)



查看旧证书信息：

```bash
# openssl x509 -in xxx.crt -text -noout 
```



浏览器中查看证书信息

打开调试窗口

![企业微信截图_20201116140253](C:\Users\ubt\Pictures\企业微信截图_20201116140253.png)





## 二、替换证书

2.1、将nginx中找到涉及g2证书的配置注释掉

![企业微信截图_20201116140543.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gkqzboxhptj30tt0l0wfi.jpg)



2.2、替换证书

将新证书上传到对应证书路径，且删除旧证书，将新证书重命名为旧证书名称

```bash
  $  mv 47xxx90__xxxxxxx.com.key xxxxxxx.key
  $  mv 47xxx90__xxxxxxx.com.pem xxxxxxx_new.crt
```



2.3、重新加载nginx配置

```bash
# nginx -t
# nginx -s reload
```



2.4、在浏览器中确认新证书已生效