## 一、现象

使用securecrt连接ubuntu ，报

```bash
No compatible key exchange method. The server supports these methods: curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group14-sha256
```



## 二、解决办法

修改sshd_config文件

```bash
$ sudo vi /etc/ssh/sshd_config
# 添加以下内容
KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1,diffie-hellman-group-exchang
e-sha1,diffie-hellman-group1-sha1

$ sudo systemctl restart sshd
```

