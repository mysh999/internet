1、查看grub配置

```bash
#  cat /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto spectre_v2=retpoline rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```





2、查看顺序

```bash
# awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2-efi.cfg
0 : CentOS Linux (3.10.0-1062.el7.x86_64) 7 (Core)
1 : CentOS Linux (0-rescue-12d12b586ac140ee8c7b001cc566ed2c) 7 (Core)
2 : Windows Boot Manager (on /dev/nvme0n1p1)
```



3、查看当前

```bash
# grub2-editenv list
saved_entry=CentOS Linux (3.10.0-1062.el7.x86_64) 7 (Core)
```



4、修改

```bash
# grub2-set-default 2

[root@localhost ~]# grub2-editenv list
```





5、重启

```bash
# reboot
```

