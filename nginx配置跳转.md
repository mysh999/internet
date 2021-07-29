## 一、需求说明

访问

```html
https://xxxx.xxxxx.com/v2/collect-rest/sign?appId=100010011&deviceId=TestUBT001
```





需要在nginx上配置以下反代

由于/v2/collect-rest不是工程的上下文，工程的上下文是/v1/collect-rest，所以需要配置访问/v2/collect-rest的时候，自动跳转到/v1/collect-rest





## 二、解决办法

```bash
upstream v1-realtime {server 172.31.0.51:xxxx;}


        location /v2/collect-rest/sign {
                        rewrite  ^/v2/(.*)$ /v1/$1 break;    # 配置跳转

                                        proxy_set_header Host $host;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_pass http://v1-realtime;
        }

```

