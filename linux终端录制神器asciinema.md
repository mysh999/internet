## 一、asciinema说明

asciinema是一款开源的终端录制工具，可以将命令行中的输入输出保存在文件中，也提供在终端或者web浏览器中进行回放。类似堡垒机中的录像功能。



## 二、部署

可以直接使用apt-get、yum或者pip进行安装

安装

```bash
 $ sudo apt-get install asciinema
```



查看版本

```bash
$ sudo asciinema --version
asciinema 2.0.0
```



## 三、使用

录制

```bash
$ sudo asciinema rec 111.cast
```

按CTRL+D终止录制



播放

```bash
$ sudo asciinema play 111.cast
```



如果想网上观看和分享

上传

```bash
$ sudo  asciinema upload 111.cast
View the recording at:

    https://asciinema.org/a/yYZrpiqzY5y6XBnMub7q56NoW

This installation of asciinema recorder hasn't been linked to any asciinema.org
account. All unclaimed recordings (from unknown installations like this one)
are automatically archived 7 days after upload.

If you want to preserve all recordings made on this machine, connect this
installation with asciinema.org account by opening the following link:

    https://asciinema.org/connect/03175912-1ae4-4a8c-b07b-011e29334655
```

使用以下地址播放

```html
https://asciinema.org/a/yYZrpiqzY5y6XBnMub7q56NoW
```



也可以在本地通过 url播放

```bash
$ sudo asciinema play https://asciinema.org/a/yYZrpiqzY5y6XBnMub7q56NoW
```

