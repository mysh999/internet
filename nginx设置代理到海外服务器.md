## 一、需求

由于墙的原因，国内访问官方翻译接口不通

接口如下:

```htm
https://synthesis-service.scratch.mit.edu/synth?locale=cmn-CN&gender=female&text=%E4%BD%A0%E5%A5%BD
```





## 二、解决办法

在一台新加坡（或者香港）的服务器上，配置某域名nginx如下：

```bash
... ...
        location ^~ /synth {
            proxy_pass https://synthesis-service.scratch.mit.edu;
            # proxy_redirect off;
            # proxy_store off;
            # proxy_set_header Host $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
```





## 三、验证

访问以下接口

```html
 https://edu-southeast.xxxxxxxxxxxx.com/synth?locale=cmn-CN&gender=female&text=%E4%BD%A0%E5%A5%BD 
```

接口通畅