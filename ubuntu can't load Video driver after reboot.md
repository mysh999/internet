## 一、problems

after reboot ubuntu 18，the installed nvidia driver can't load ,tips is the drive is not install coreccet



## 二、resoleve way

2.1、download the nvidia drive soft



2.2、switch the command console

press ctrl+alt+F1 key to enter command console



2.3、kill X server

```bash
# ps -ef|grep X
# sudo service lightdm stop
```



2.4、reinstall driver

```bash
# sudo ./NVIDIA-XXX
```



2.5、reboot os



2.6、disable auto soft update

```bash
# sudo vi /etc/apt/apt.conf.d/10periodic
#０是关闭，1是开启，将所有值改为0

APT::Periodic::Update-Package-Lists "0";

APT::Periodic::Download-Upgradeable-Packages "0";

APT::Periodic::AutocleanInterval "0";
```

