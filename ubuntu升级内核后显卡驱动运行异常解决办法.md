## 一、现象

ubuntu 18.04系统，安装nvidia显卡驱动

在系统做了内核升级后，执行 nvidia-smi命令异常，如下输出 

```bash
# nvidia-smi

NVIDIA-SMI has failed because it couldn’t communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running

```







## 二、解决办法

重新安装驱动



### 2.1、查看推荐版本

```bash
# ubuntu-drivers devices
```

![企业微信截图_20210421175330.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gprikli4zhj30vd08aqpq.jpg)

可知推荐的版本是nvidia-driver-460



### 2.2、添加仓库

```bash
# add-apt-repository ppa:graphics-drivers/ppa
# apt update   
# 更新索引
```



### 2.3、安装驱动

```bash
# apt install nvidia-driver-460
```

再次输入nvidia-smi正常输出（多运行几次）