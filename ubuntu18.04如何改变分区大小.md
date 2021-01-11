说明：

ubuntu18.04版本，采用的ext4分区系统（非lvm）

由于根分区划分不合理，需要变更大小，从别的分区分配一些空间给根分区





操作说明：

1、先安装gparted包

```bash
$ sudo apt-get install gparted
```



2、准备同版本的ubuntu iso文件



3、安装usb-creator-gtk包（制作ubuntu 启动盘）

```bash
# sudo apt-get install usb-creator-gtk
```

执行usb-creator-gtk命令，使用ubtuntu iso文件和U盘制作启动盘，再将U盘插入到电脑中，重启ubuntu系统

进入试用模式（try ubuntu without install）



4、执行gparted命令



5、变更分区大小

压缩好以后，然后选择根目录，resize/move，把刚才压缩出来的空间增加到根目录下。

最后保存更改：即选择菜单栏 >> Edit >> Apply all Operations >> Apply。
重启电脑，拔掉u盘，即可扩容成功。

作完成时有两个warning，不用理会，重启系统后一切正常