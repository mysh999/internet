1、安装包

```bash
# yum -y install rpcbind nfs-utils
```



# 

2、配置nfs

```bash
# vi /etc/exports 
/dfs 10.8.73.0/24(rw,sync,no_root_squash)
```





3、配置/dfs让客户端普通用户可以读写

```bash
# chmod -R o+w /dfs/ 
```





4、查看nfs服务器挂载情况

```bash
# showmount -e localhost 
```





启动rpc服务

```bash
# systemctl restart rpcbind.service 
```



启动NFS服务

```bash
# systemctl start nfs.service 
```





查看状态

```bash
# systemctl status nfs.service 
```





配置NFS开机自启动

```bash
# chkconfig rpcbind on
# chkconfig nfs on
# chkconfig --list rpcbind
# chkconfig --list nfs 
```



5、重新加载nfs配置

```bash
# exportfs -rv 
```





6、客户端配置：
挂载测试

```bash
# mount -t nfs 10.8.73.133:/dfs /dfs
```



# 

7、开机自启动

```bash
#vi /etc/fstab
10.8.73.133:/dfs   /dfs     nfs    defaults     0  0
```



