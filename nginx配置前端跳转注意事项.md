## 一、需求说明

前端配置了跳转路由

![企业微信截图_16038673291014.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gk56htg3s8j309d016a9u.jpg)



nginx也需要做相应配置





## 二、nginx配置注意事项

```bash
# cat cbis-sg.xxxxxxxxxxxxxxxxxxxxxxxxxx.com.conf
    server {
        listen       80;
        server_name  cbis-sg.xxxxxxxxxxxxxxxxxxxxxxxxxx.com;
        
        access_log   /var/log/nginx/cbis-sg.xxxxxxxxxxxxxxxxxxxxxxxxxx.com-access.log main;
        error_log    /var/log/nginx/cbis-sg.xxxxxxxxxxxxxxxxxxxxxxxxxx.com-error.log;
        rewrite ^/webproject/cbis/(.*)$  https://cbis-sg.xxxxxxxxxxxxxxxxxxxxxxxxxx.com/$1 permanent;    #加配这里
        
        
        ... ...
        
                location / {
  
            root  /usr/share/nginx/html/webproject/cbis;    #指向实际路径
            index index.html;
            client_max_body_size  1000m;
            # return 403;
        }
```







## 三、跳转效果

浏览器中输入

```html
https://cbis-sg.xxxxxxxxxxxxxxxxxxxxxxxxxx.com/
```



会自动跳转到

```bash
https://cbis-sg.xxxxxxxxxxxxxxxxxxxxxxxxxx.com/#/webproject/cbis/user/login?redirect=https://cbis-sg.xxxxxxxxxxxxxxxxxxxxxxxxxx.com/#/
```

