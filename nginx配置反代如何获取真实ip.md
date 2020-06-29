如果想要通过X-Forwarded-For获得用户ip，就必须先使用proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for; 这样就可以获得用户真实ip。

```
        location /operation_service/ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://operation-service;
        }
```