1、查看网络设备

```bash
# nmcli dev
DEVICE          TYPE      STATE         CONNECTION 
wlp4s0          wifi      connected     UBT-Users  
p2p-dev-wlp4s0  wifi-p2p  disconnected  --         
enp0s31f6       ethernet  unavailable   --        
```



2、启用wifi

```bash
# nmcli r wifi on
```



3、扫描wifi

```bash
# nmcli dev wifi
```



4、连接wifi

```bash
# nmcli dev wifi connect "wifi名" password "密码"
```

