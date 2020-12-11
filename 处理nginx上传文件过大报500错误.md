## 一、现象

在操作系统做了内核升级后，前端页面上传附件，超过1M报500内部错误

![企业微信截图_20201211165632.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glk0ra1gpyj311c0k2wg3.jpg)



## 二、解决办法

### 2.1、检查nginx配置文件

```bash
#vi ./conf.d/xxx.conf
server {
        listen       80;
        server_name  cbis.ubtrobot.com;
        client_max_body_size  1000m;    #新增这一行
        ... ...
       }      
```

说明：

client_max_body_size 参数可以配置在nginx.conf文件中，也可以配置在子配置文件下的server或者location下

范围由大到小，如果二者或者多者都有配置，范围小的生效。

官方说明：

```html
https://nginx.org/en/docs/http/ngx_http_core_module.html?&_ga=2.160768948.842148723.1607677269-317095112.1607677269#client_max_body_size

# 内容如下：
Syntax:	client_max_body_size size;
Default:	
client_max_body_size 1m;
Context:	http, server, location
```



### 2.2、检查文件属主

```bash
# ll /var/lib|grep -i nginx
drwxrwx---   3 nginx   root    4096 Nov  1 10:02 nginx

# 修改为nobody属主
# chown -R nobody:nobody /var/lib/nginx
```

