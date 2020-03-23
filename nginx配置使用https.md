## 一、原理

HTTP协议以明文方式发送内容，不提供任何方式的数据加密。为了数据传输的安全，HTTPS在HTTP的基础上加入了SSL协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密



nginx使用的就是PEM格式的证书,将其拆分开就是需要两个文件,一个是.key文件,一个是.crt文件.



## 二、申请证书

可以通过云平台申请，也可以linux自己生成，这里采用自己生成的方式



## 三、生成证书

3.1、生成密钥key

```bash
# openssl genrsa -des3 -out server.key 2048
Generating RSA private key, 2048 bit long modulus
..........................................................................................................................................+++
.................+++
e is 65537 (0x10001)
Enter pass phrase for server.key:
Verifying - Enter pass phrase for server.key:
# 会有2次要求输入密码，回车即可
```



如果要去除密码（可选）

```bash
# openssl rsa -in server.key -out server.key
```





3.2、创建证书的申请文件server.csr

```bash
# openssl req -new -key server.key -out server.csr
Enter pass phrase for server.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```



3.3、创建ca证书

```bash
# openssl req -new -x509 -key server.key -out ca.crt -days 3650
Enter pass phrase for server.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:
```

得到一个ca.crt的证书,这个证书用来给自己的证书签名



3.4、创建有效期10年的服务器证书server.crt

```bash
# openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
Signature ok
subject=/C=CN/L=Default City/O=Default Company Ltd
Getting CA Private Key
Enter pass phrase for server.key:
```



至此，一共生成5个文件，其中server.crt和server.key就是nginx需要的证书文件

```bash
# ls -lt
-rw-r--r--  1 root root 1103 3月  23 14:56 server.crt
-rw-r--r--  1 root root   17 3月  23 14:56 ca.srl
-rw-r--r--  1 root root 1220 3月  23 14:54 ca.crt
-rw-r--r--  1 root root  952 3月  23 14:53 server.csr
-rw-r--r--  1 root root 1751 3月  23 14:50 server.key
```



## 四、安装nginx

4.1、安装nginx

```bash
# yum -y install nginx
```



4.2、启动和设置开机自启动

```bash
# systemctl start nginx.service
# systemctl enable nginx.service
```



4.3、查看nginx状态

```bash
# nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-stream_ssl_preread_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-http_auth_request_module --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
```



## 五、配置nginx

5.1、配置nginx

```bash
# cat /etc/nginx/conf.d/test.conf 
        server
    {
        listen 443 ssl;
        server_name test.ubtrobot.com;
        root /usr/share/nginx/html;
        access_log  /var/log/nginx/access.log main;
        error_log  /var/log/nginx/error.log;

        ssl_certificate      /root/keys/server.crt;
        ssl_certificate_key  /root/keys/server.key;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

        ssl_session_cache    shared:SSLCache:10m;
        ssl_session_timeout  10m;
        resolver 8.8.8.8 8.8.4.4  valid=300s;
        resolver_timeout 10s;
        }
```



5.2、重启服务

```bash
# systemctl restart nginx.service
```





## 六、验证

6.1、修改本地电脑的hosts文件

```bash
IP   test.ubtrobot.com
```



6.2、在浏览器中输入

https://test.ubtrobot.com

会弹出“您的连接不是私密连接”，不用理会，点继续即可访问

