问题描述：

访问以下工程的swagger接口，加载不出界面

https://aaa-na.xxxxx.com/v1/aaa/swagger/index.html



打开调试模式，显示

https://aaa-na.xxxxx.com/v1/aaa/swagger/swagger-ui-bundle.js

和

https://aaa-na.xxxxx.com/v1/aaa/swagger/swagger-ui-standalone-preset.js

文件状态failed



解决思路：

1、直接打开工程服务器IP，可以打开

http://ip:端口/v1/aaa/swagger/index.html



2、查看nginx日志，发现报权限问题

```bash
# tail -f /var/log/nginx/aaa-na.xxxxx.com-error.log
2021/12/08 14:35:35 [crit] 4161#4161: *7672831 open() "/var/lib/nginx/tmp/proxy/1/21/0000000211" failed (13: Permission denied) while reading upstream, client: 103.220.9.173, server: aaa-na.xxxxx.com, request: "GET /v1/aaa/swagger/swagger-ui-standalone-preset.js HTTP/1.1", upstream: "http://172.31.0.27:9090/v1/aaa/swagger/swagger-ui-standalone-preset.js", host: "aaa-na.xxxxx.com", referrer: "https://aaa-na.xxxxx.com/v1/aaa/swagger/index.html"
```

因为nginx配置里是nobody用户



3、解决办法

```bash
# cd /var/lib/
# chown nobody.nobody nginx/ -R 
```

再次打开

https://aaa-na.xxxxx.com/v1/aaa/swagger/index.html

界面加载正常