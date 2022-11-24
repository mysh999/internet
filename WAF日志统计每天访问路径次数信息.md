## 一、需求

在waf中统计每天对应接口path请求次数信息



## 二、语句

```sql
request_path: /v1/opus/14526/like | SELECT host,real_client_ip,request_path,COUNT(*) as num  GROUP by  host,real_client_ip,request_path order by num desc
```

