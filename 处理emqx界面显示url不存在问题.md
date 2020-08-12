## 一、问题描述

emqx容器化集群，打开节点1的dashboard，显示URL not found

![企业微信截图_15972032257297.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghnvvvz6psj30sd0ls756.jpg)





## 二、解决办法

原因是端口1883被占用，



修改端口：

```bash
[root@ plugins]# vi emqx_management.conf
[root@ plugins]# docker-compose down;docker-compose up -d --build;docker-compose logs -f
```





![企业微信截图_15972037425837.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ghnvw8xeerj30jb0mlaaw.jpg)