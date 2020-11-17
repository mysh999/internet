## 一、问题描述

执行nginx -t提示

```bash
# nginx -t
nginx: [emerg] unknown directive "stream" in /etc/nginx/nginx.conf:73
nginx: configuration file /etc/nginx/nginx.conf test failed
```





## 二、解决办法

需要手动加载stream模块

在nginx.conf配置文件第一行加入

```bash
# vi nginx.conf
load_module /usr/lib64/nginx/modules/ngx_stream_module.so;   #第一行

# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

问题解决