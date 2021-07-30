## 一、需求说明

需求一：在nginx全局中配置禁用缓存

需求二：在index.html文件中配置禁用缓存



## 二、解决办法

需求一：

```bash
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css)$ {
 #禁止缓存，每次都从服务器请求
  add_header Cache-Control no-store;
}
```







需求二：

在html头文件中添加

```bash
<meta http-equiv="pragma" content="no-cache">
<meta http-equiv="cache-control" content="no-cache">
<meta http-equiv="expires" content="0">   
```

