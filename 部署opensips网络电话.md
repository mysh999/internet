## 一、opensips说明

opensips是一款功能强大的视频会议和电话软件，使用app用户可以在线电话、会议，支持多人在线交流 



## 二、环境说明

准备一台Linux服务器，环境如下：

- Centos7.7

- mysql5.7

- 关闭selinux和防火墙

- 准备软件:

  `https:``//opensips.org/pub/opensips/2.2.8/`

- 安装必要包

  ```bash
  # yum -y install gcc
  # yum -y install bison
  # yum -y install flex
  # yum install http://mirrors.ubtrobot.com/mysql/yum/mysql56-community-el7/mysql-com
  munity-devel-5.6.37-2.el7.x86_64.rpm
  ```

  



## 三、安装配置

3.1、解压

```bash
# tar -zxvf opensips-2.2.8.tar.gz
```



3.2、进入解压目录，执行make menuconfig

```bash
# make menuconfig
```

![企业微信截图_20200427164544.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ge8f5iprzjj30dn03kq3b.jpg)

选进入Configure compile Options，选择加载mysql模块

![企业微信截图_20200427164642.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ge8f6cphaxj30ck0b5q46.jpg)



按键盘Q，退出保存！--执行：compile And intsall opensips --等待安装执行完毕



安装执行完毕，回车，Exit & save all changes 查看是否安装ok



3.3、配置

```bash
# vi /usr/local/etc/opensips/opensipsctlrc
```

![企业微信截图_20200427171038.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ge8fvam6bsj30mg0ikte1.jpg)



3.4、执行脚本

```bash
# cd /usr/local/sbin/
# ls
opensips  opensipsctl  opensipsdbctl  opensipsunix  osipsconfig  osipsconsole
#  ./opensipsdbctl  create
```

默认全部加载56张表



3.5、启动测试

```bash
[root@os sbin]# ./opensipsctl start
```

![企业微信截图_20200427171235.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ge8fx9s85fj30cq02rq3a.jpg)



3.6、添加用户

```bash
[root@os sbin]# ./opensipsctl add 1000 123456    #1000是用户，123456是密码
new user '1000' added
[root@os sbin]# ./opensipsctl add 1001 123456
new user '1001' added
```



## 四、客户端安装

安卓手机安装以下软件：

http://js.xiazaicc.com/anzhuoyy/linphone_downcc.apk



windows电脑安装以下软件：

http://www.linphone.org/sites/default/files/linphone-4.1.1-win32.exe



安装后配置

![企业微信截图_20200427171618.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ge8g12hx2jj30ba0pamy8.jpg)

![企业微信截图_20200427171703.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ge8g1zdpizj30b60p2t9s.jpg)



## 五、拨打电话测试

比如两部手机安装安卓版本，且都连到同一网络

互拨对方号码可通信

![企业微信截图_20200427171959.png](http://ww1.sinaimg.cn/large/007Xg1efgy1ge8g4wz5oaj30b50l2jvw.jpg)