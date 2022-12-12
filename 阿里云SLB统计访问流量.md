## 一、统计语句 

```sql
*|select host,request_uri,sum(body_bytes_sent) /1024 /1024 as "流量 MB",count(*) as "访问次数"  group by host,request_uri order by "流量 MB"  desc
```





## 二、效果

![企业微信截图_20221212152158.png](http://tva1.sinaimg.cn/large/007Xg1efgy1h911ztpyo6j30z00o7n9s.jpg)