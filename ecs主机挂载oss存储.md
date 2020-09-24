1、功能需求

在多台阿里云ECS主机上挂载oss 对象存储bucket



2、shell部署示例

```bash
# vi mount_oss.sh
#!/usr/bin/sh
export CONFIG_OSS=/etc/passwd-ossfs


function install_ossfs(){
rpm -qa|grep -i ossfs
if [[ $? -eq 0 ]];then
	echo "have installed ossfs package"
	exit 0
else
	yum install http://gosspublic.alicdn.com/ossfs/ossfs_1.80.6_centos7.0_x86_64.rpm -y
	ln -s /usr/local/bin/ossfs /usr/bin
fi
}

function config_ossfs(){
cat $CONFIG_OSS|grep -i ubt-backup
if [[ $? -eq 0 ]];then
    echo "have configure ossfs"
    exit 0
else
    echo 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' > $CONFIG_OSS
    chmod 0640 $CONFIG_OSS
    mkdir -p /data/oss
    ossfs ubt-backup /data/oss -ourl=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.aliyuncs.com -oallow_other -omp_umask=002
fi
}

function show_ossfs(){
df -h|grep -i ossfs
if [[ $? -eq 0 ]];then
	echo "have mount ossfs"
	exit 0
else	
	echo "no mount ossfs,Please mount again!!!"
fi
}


function main () {
    install_ossfs
    config_ossfs
    show_ossfs
}

main


```

