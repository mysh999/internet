## 一、当前环境

```bash
# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root   50G  4.1G   46G   9% /
devtmpfs              32G     0   32G   0% /dev
tmpfs                 32G     0   32G   0% /dev/shm
tmpfs                 32G  8.6M   32G   1% /run
tmpfs                 32G     0   32G   0% /sys/fs/cgroup
/dev/vda1           1014M  159M  856M  16% /boot
/dev/mapper/cl-data  1.9T  987M  1.9T   1% /data
overlay               50G  4.1G   46G   9% /var/lib/docker/overlay2/1e72e4a95e5dccc4688fb0fa2cd309fb787f5f90641b2fa1dc849f2d8b29ad98/merged
overlay               50G  4.1G   46G   9% /var/lib/docker/overlay2/ae4acf0b8e6c9620a1b7594c0e892b91cd98a7a69713bc3b02ff2d70a4c5697d/merged
overlay               50G  4.1G   46G   9% /var/lib/docker/overlay2/864c672710e59851ec2152108dca20065c68e5ad9c2f6a7e5a869ea459f381b9/merged
shm                   64M     0   64M   0% /var/lib/docker/containers/df87bede948446d83f0db8af69ed4d97fdc94e12b66e74da1d88158c4cf25939/mounts/shm
shm                   64M     0   64M   0% /var/lib/docker/containers/9d1aa3c2086e405bbd13d78d1e3247944c92199d1b1ab00f92af64e87158899b/mounts/shm
shm                   64M     0   64M   0% /var/lib/docker/containers/d3f69a784ac9740fe5952ea0c259c95ef6d3e4661a8d3c8fd51bf9010db6e666/mounts/shm
tmpfs                6.3G     0  6.3G   0% /run/user/0

# lsblk 
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1  4.1G  0 rom  
vda         252:0    0    2T  0 disk 
├─vda1      252:1    0    1G  0 part /boot
└─vda2      252:2    0    2T  0 part 
  ├─cl-root 253:0    0   50G  0 lvm  /
  ├─cl-swap 253:1    0 31.5G  0 lvm  [SWAP]
  └─cl-data 253:2    0  1.9T  0 lvm  /data
  
  
# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Mon Dec  2 14:00:31 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=a2d1b940-caa2-4785-90c1-50f845e3a6cf /boot                   xfs     defaults        0 0
/dev/mapper/cl-data     /data                   xfs     defaults        0 0
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
```





## 二、操作记录

2.1、将/data目录下的文件拷贝到其他地方



2.2、卸载/data

```bash
# umount /data
# fuser -mk /data 
```



2.3、删除lv

```bash
# lvremove /dev/cl/data
Do you really want to remove active logical volume cl/data? [y/n]: y
  Logical volume "data" successfully removed
```



2.4、扩展root lv

```bash
# lvresize -L 500G /dev/cl/root
  Size of logical volume cl/root changed from 50.00 GiB (12800 extents) to 500.00 GiB (128000 extents).
  Logical volume cl/root successfully resized.
```

2.5、扩展/文件系统

```bash
# xfs_growfs /
meta-data=/dev/mapper/cl-root    isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 13107200 to 131072000
```



2.6、重新创建data lv

```bash
# lvcreate -L 1.43t -n data cl
  Rounding up size to full physical extent 1.43 TiB
  Logical volume "data" created.
```

2.7、格式化

```bash
# mkfs.xfs /dev/mapper/cl-data 
```



2.8、挂载

因为挂载点和lv名称不变，所以直接执行挂载

```bash
[root@localhost /]# mount -a
[root@localhost /]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root  500G  5.0G  496G   1% /
devtmpfs              32G     0   32G   0% /dev
tmpfs                 32G     0   32G   0% /dev/shm
tmpfs                 32G   17M   32G   1% /run
tmpfs                 32G     0   32G   0% /sys/fs/cgroup
/dev/vda1           1014M  159M  856M  16% /boot
tmpfs                6.3G     0  6.3G   0% /run/user/0
/dev/mapper/cl-data  1.5T   33M  1.5T   1% /data
```



2.9、查看扩容效果

```bash
# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root  500G  4.1G  496G   1% /
devtmpfs              32G     0   32G   0% /dev
tmpfs                 32G     0   32G   0% /dev/shm
tmpfs                 32G   17M   32G   1% /run
tmpfs                 32G     0   32G   0% /sys/fs/cgroup
/dev/vda1           1014M  159M  856M  16% /boot
tmpfs                6.3G     0  6.3G   0% /run/user/0
/dev/mapper/cl-data  1.5T  995M  1.5T   1% /data
```

