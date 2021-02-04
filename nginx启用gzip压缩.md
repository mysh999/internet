## 一、概念说明

nginx主配置文件可以配置gzip压缩功能

虚拟主机配置文件无需再配置gzip功能，以nginx.conf配置文件为准，否则容易配置冲突



nginx.conf文件添加以下配置:

```bash
    # gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript text/x-component application/json application/javascript application/x-javascript application/xml application/xhtml+xml app
lication/rss+xml application/atom+xml application/x-font-ttf application/vnd.ms-fontobject image/svg+xml image/x-icon font/opentype;
```





## 二、验证gzip功能生效

使用chrome浏览器，通过F12打开调试窗口

![企业微信截图_20210203200705.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gnalqxme40j30o70b7q3x.jpg)

显示gzip字样，说明启用了gzip功能