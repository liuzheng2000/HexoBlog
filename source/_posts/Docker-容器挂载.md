---
title: Docker 容器挂载
abbrlink: 1612373770
date: 2021-11-29 09:14:23
tags:
  - Docker
categories: Docker
description: Docker 挂载命令解析
---
**Docker 容器挂载**

```
// 运行容器并挂载命令
docker run --name mynginx  //容器起名为nginx
-d //后台运行 
-p 80:80 //把主机80端口映射到容器80端口
--restart=always 
--privileged=true //防止挂载时权限不够 
-v /export/nginx/conf/nginx.conf:/etc/nginx/nginx.conf
-v /export/nginx/html:/etc/nginx/html 
-v /export/nginx/log:/var/log/nginx 
//把文件挂载到主机目，主机文件目录:容器文件目录 
nginx //运行镜像的名称REPOSTITORY
```

```
docker run --name my-nginx -p 80:80 -v /export/www/nginx/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
```

