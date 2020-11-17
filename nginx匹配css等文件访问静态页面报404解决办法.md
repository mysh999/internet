## 一、问题描述

nginx原先访问前端正常，

变更nginx 443配置如下后：

```bash

        # assets, media
        location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
            expires 7d;
            access_log off;
        }

        # svg, fonts
        location ~* \.(?:svgz?|ttf|ttc|otf|eot|woff2?)$ {
            add_header Access-Control-Allow-Origin "*";
            expires 7d;
            access_log off;
        }
```



访问前端页面就报404错误



## 二、解决办法

在server段内添加

```bash
      access_log           /var/log/nginx/test79.ubtrobot.com-access.log main;
      error_log           /var/log/nginx/test79.ubtrobot.com-error.log;
      # root
      root           /data/www;               #新增
```

再次访问恢复正常