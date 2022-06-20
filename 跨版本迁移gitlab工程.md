## 一、需求说明

将以下工程

```bash
https://10.10.1.34/soft-depart/abcchinc_alpha2_service
```

迁移到

```bash
https://gitlab.abc.com/ecbg/cloudsoft/abcchinc_alpha2_service
```





## 二、在34旧服务器上操作

2.1、在34上配置一个用户，对该工程具有master权限（如果有用户可忽略）



2.2、配置git客户端

windows下可以在cmder下配置，不需要额外安装git客户端

```bash
$ git config --global user.name "mysh"
$ git config --global user.email "mysh@qq.com"
```



2,3、在旧服务器上配置ssh密钥



2.4、下载工程文件

```bash
$ git clone git@10.10.1.34:soft-depart/abcchinc_alpha2_service.git
$ cd ubcchinc_alpha2_service
```



2.5、清理旧站点

```bash
$ git remote -v
origin  git@10.10.1.34:soft-depart/abcchinc_alpha2_service.git (fetch)
origin  git@10.10.1.34:soft-depart/abcchinc_alpha2_service.git (push)

$ git remote remove origin

$ git remote -v    #为空
```



## 三、在gitlab新站点执行

3.1、在新站点上创建同名用户



3.2、将下载的工程添加进缓存区

```bash
$ git add  -A

$ git commit -m "init2"
```



3.3、新增远程站点

```bash
$ git remote add origin git@gitlab.abc.com:mysh/abcchinc_alpha2_service.git

$ git remote -v
origin  git@gitlab.abc.com:mysh/abcchinc_alpha2_service.git (fetch)
origin  git@gitlab.abc.com:mysh/abcchinc_alpha2_service.git (push)

```



3.4、推送

```bash
$ git push -u origin master
```




