---
title: Ubuntu安装Docker并启动Nginx
abbrlink: 3003506474
date: 2021-11-29 09:16:50
tags:
  - Docker
categories: Docker
description: Ubuntu使用Docker简单记录
---
**Ubuntu安装Docker并启动Nginx**

Ubuntu 安装docker

```bash
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]

```

```bash
sudo usermod -aG docker ubuntu   #docker命令授权
```

### Docker安装nginx

```nginx
docker pull nginx
```

简单运行

```bash
docker run --name my-nginx -p 80:80 -d nginx
docker ps | grep my-nginx
```

![image-20211125160341793](http://typa.qingyun.run/img/image-20211125160341793.png?x-oss-process=style/qingyun)

```
IP:80
```

最基础的配置

### 云服务器安装node + hexo

```bash
sudo apt install nodejs
```

![image-20211125160841079](http://typa.qingyun.run/img/image-20211125160841079.png?x-oss-process=style/qingyun)

```bash
sudo apt install npm
```

![image-20211125161855785](http://typa.qingyun.run/img/image-20211125161855785.png?x-oss-process=style/qingyun)

```bash
npm config set registry https://registry.npm.taobao.org
npm config get registry
```

```bash
npm install -g hexo-cli
```

