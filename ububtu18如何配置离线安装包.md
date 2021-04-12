## 一、本地下载deb安装包

如果要下载nginx、docker安装包

```bash
#  apt-get -dy install nginx
#  apt-get -dy install docker.io
```

文件下载到 /var/cache/apt/archives/ 目录中



## 二、将下载文件拷贝到指定目录

```bash
# cp -rf /var/cache/apt/archives/*.deb /opt/offlinepkg/
# chmod -R 777 /opt/offlinepkg/
```



### 三、创建索引

```bash
# dpkg-scanpackages /opt/offlinepkg/ /dev/null |gzip >/opt/offlinepkg/Packages.gz
```

如果出现以下警告信息可以忽略

```bash
dpkg-scanpackages: warning: Packages in archive but missing from override file:
dpkg-scanpackages: warning:   bridge-utils cgroupfs-mount containerd docker.io fontconfig-config fonts-dejavu-core libfontconfig1 libgd3 libjbig0 libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libtiff5 libwebp6 libxpm4 nginx nginx-common nginx-core pigz runc ubuntu-fan
dpkg-scanpackages: info: Wrote 25 entries to output Packages file.
```





## 四、配置本地文件源

```bash
# vi /etc/apt/sources.list
deb file:// /opt/offlinepkg/ 
```





## 五、更新索引

```bash
# apt-get update --allow-insecure-repositories 
```

会有一些警告信息，忽略



## 六、执行安装包

```bash
# apt-get -y --force-yes install docker.io
# apt-get -y --force-yes install nginx
```

