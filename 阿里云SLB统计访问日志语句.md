## 1、统计访问流量

```bash
*|select host,request_uri,sum(body_bytes_sent) /1024 /1024 as "流量 MB",count(*) as "访问次数"  group by host,request_uri order by "流量 MB"  desc


host: apis.abc.com|select request_uri,COUNT(*) as "次数", sum(body_bytes_sent)/ 1024 / 1024 as "下载流量MB" GROUP by request_uri order by "下载流量MB" desc  
```



## 2、统计报错信息



匹配返回码

```bash
status =502|select host as "域名",request_uri as "请求地址",status as "响应状态码",request_time order by request_time desc 
```



```bash
status >400 |select host as "域名",request_uri as "请求地址",status as "响应状态码",request_time order by request_time desc 
```



根据uri匹配错误次数

```bash
status=502 and request_uri: /v1/upgrade*|select host as "域名",url_extract_path(request_uri) as "上下文",COUNT(*) as errorCount GROUP BY "域名","上下文"
```



大于400匹配错误次数

```bash
status >=400|select host as "域名",request_uri as "请求地址",status as "响应状态码",COUNT(*) as "次数" group by host,request_uri,status order by "次数" desc
```







## 3、匹配后端主机查询

```bash
upstream_addr=172.31.0.26* and status >400 |select host,request_uri,status
```

