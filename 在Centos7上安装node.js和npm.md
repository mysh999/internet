参考：

https://www.myfreax.com/how-to-install-node-js-on-centos-7/





## 一、概念说明

Node.js：  是一个跨平台的JavaScript运行时环境，允许服务器端执行JavaScript代码。 Node.js主要用于后端，但也作为全栈和前端解决方案而流行

npm:         Node Package Manager的缩写，是Node.js的默认软件包管理器，也是全球最大的开源Node.js软件包的发布软件仓库





## 二、安装步骤

### 2.1、添加NodeSource yum存储库

    ```bash
# curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -
    ```



### 2.2、安装node.js

```bash
# yum install nodejs
```



### 2.3、验证版本

```bash
# node --version    
v10.24.1

# npm --version
6.14.12
```

