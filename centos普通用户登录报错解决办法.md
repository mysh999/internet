## 一、问题描述

使用普通用户登录centos时，登录不了

使用root用户登录，再切换到普通用户，提示如下：

```bash
# su - runner
上一次登录：四 10月 15 10:53:53 CST 2020从 10.10.30.114pts/4 上
su: failed to execute /bin/bash: 资源暂时不可用
```





## 二、解决办法

原因：普通用户的资源限制未打开

```bash
#  egrep -v "^$|^#" /etc/security/limits.d/20-nproc.conf
*          soft    nproc     4096     #默认数值
root       soft    nproc     unlimited
```



解决办法：

```bash
# vi /etc/security/limits.d/20-nproc.conf
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

*          soft    nproc     unlimited        #修改 
root       soft    nproc     unlimited
```

再次登录，恢复正常