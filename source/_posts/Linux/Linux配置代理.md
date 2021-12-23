---
title: Linux配置代理
date: 2021-12-23 16:20:00
tags: Linux
categories: Linux
---
## 背景

linux 啥都好，就是日常使用的软件不够丰富，就比如 clash: windows 和 mac 上都有专门的图形化界面，而 linux  上就没有，只有简陋的命令行。使用自建 vps 倒也罢了，如果你使用的是机场，那么订阅规则总不可能自己一个个地手动去写吧？

最近我的 vps 故障了，于是就试用了几天机场，虽然对它的安全性抱有顾虑，但它的速度实在是太香了，那么我们有没有什么办法愉快地在 linux 平台上管理我们的机场订阅呢？

简单的方法是有的，据我所知，目前惟一一个支持图形化界面自动管理订阅的软件是：[QV2ray](https://github.com/Qv2ray/Qv2ray) ，通过它你能很方便地管理 ssr,v2ray 的订阅规则。QV2ray 的本体并不支持 trojan 规则，好在官方提供了一个 [QvPlugin-Trojan](https://github.com/Qv2ray/QvPlugin-Trojan) 插件，通过它你也可以很方便地管理 trojan 订阅。

它的界面是这样的：

​	[ 		![img](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-30.png) 	](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-30.png) 



那为什么还要去折腾 clash 呢？从我的经验看，主要有下面几个理由：

- clash 的分流功能更为强大，网络上有很多共享的自动代理切换规则都是基于 clash
- QV2ray 还不够成熟，使用不规则的 https 端口时会出现证书验证失败的问题，需要手动忽略证书
- 个人感觉 clash 的性能比 QV2ray 稍强

顺带一提，如果你使用的是 trojan-go 那么你也只能使用 QV2ray ，因为 clash 只支持 trojan 不支持 trojan-go.

## clash 的安装及使用

### 安装

本节简要介绍了如何在 linux 上安装 clash 。

#### 通过包管理器安装

如果你使用的是 arch ，那么可以通过 aur 很方便地安装，只要执行 `yay clash` 就可以了，这样的好处是会自动生成 service 文件，可以方便你通过 systemctl 开机启动 clash.

#### 通过 go 安装

clash 是基于 golang 实现的，因此你也可以通过 golang 编译安装，如果你使用的不是 arch ，那么你只能通过这种方式安装了。命令是 `go get -u github.com/Dreamacro/clash` ，之后会在 `~/go/bin` 目录下生成二进制文件，如果不在这个目录，那么你也可以通过 `go env` 命令查看 `gopath` 的具体位置。

通过这种方式安装的问题是你需要手动地添加 service 文件，这里我把上一步生成的 serivce 文件贴出来：

```service
[Unit]
Description=A rule based proxy in Go for %i.
After=network.target

[Service]
Type=exec
User=%i
Restart=on-abort
ExecStart=/usr/bin/clash

[Install]
WantedBy=multi-user.target
```

把这个文件放到 `/usr/lib/systemd/system/clash@.service` ，你需要修改内容是 `ExecStart` ，将其改成 clash 二进制文件所在的路径。

### 启动

在配置好 service 文件后，systemctl 的命令是：

```shell
sudo systemctl enable clash@<username> # 开机自启
sudo systemctl start clash@<username> # 启动
sudo systemctl stop clash@<username> # 停止
```

在默认情况下，clash 会监听三个端口：

- http 代理端口：7890
- socks 代理端口：7891
- clash restful 配置服务端口：9090 ，这个端口一般是提供给前端管理页面使用的

在系统中配置好相应的代理服务后就可以科学上网了。

### 配置文件

配置文件的路径是： `~/.config/clash/config.yaml` 下面生成的配置就需要贴到这个文件中。

## clash 订阅导入原理

在 linux 上导入订阅需要用到两个工具：

- [subconverter](https://github.com/tindy2013/subconverter) subconverter 的作用是将订阅连接转换成 clash 配置。
- [sub-web](https://github.com/MSKNET/sub-web) sub-web 是 subconverter 的前端，提供了一个方便的浏览器页面。

![Figure 1: 流程图](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-31.png)

Figure 1: 流程图

现在你 google clash 订阅，出来的无非就是这两个东西。 sub-web 的本质就是一个静态 html 页面，你能搜索到很多人提供的托管网站，但问题是并不安全，你并不知道他们会不会偷偷记录下你的订阅连接，所以最好的方法是本地安装。

### 本地安装 sub-web

sub-web 是一个 nodejs 项目，为了编译这个项目，你需要安装 nodejs 环境并安装 yarn 。

从 github clone 这个项目后，使用 `yarn install` ， `yarn serve` 就能启动。

### 安装 subconverter

这个比较简单，作者提供了 release ，直接下载 `subconverter_linux64.tar.gz` 文件解压后在目录下执行 `subconverter` 命令，就能启动服务，默认端口是 25500

### 使用

按照上述步骤启动 sub-web 和 subconverter 之后，访问 sub-web 的浏览器地址，这个地址在执行 `yarn server` 后会在终端提供，默认是 http://localhost:8080/ 。

使用很简单，只需要填入订阅连接和后端地址就可以了：

![img](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-32.png)

将生成的连接复制到新标签页访问，你就能得到一个完整的 clash 配置，将其复制到 `~/.config/clash/config.yaml` 文件中，重启 clash 就可以了。

## 通过 clash-dashboard 管理代理

订阅连接里一般会提供很多的代理服务器，clash 会自动选择最快的一个，有时候你或许会想要手动使用其中某个服务器，这个时候就需要用到 [clash-dashboard](https://github.com/Dreamacro/clash-dashboard) 了。

和 sub-web 一样，这也是一个前端项目，依次执行 `yarn install`, `yarn start` 就可以了，由于这个项目使用的是 vite 构建，你可能需要执行 `yarn global add vite` 安装。

~~和 sub-web 不同，clash-dashboard 并不需要 web 容器，因此你也可以通过 `yarn build` 生成静态 html 页面，然后直接在文件访问这个文件。~~ 后来我发现其实还是需要 web 容器的，奇怪，印象中以前的版本是不需要的。

界面是这样的：

​	[ 		![img](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-34.png) 	](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-34.png) 



## 通过 netlify 托管 sub-web 和 clash-dashboard

在本地使用这些服务虽然很安全，但也很麻烦，幸运的是 netlify 上提供了免费的静态网站托管服务。

### fork 项目

由于 netlify 只能托管自己的 github 项目，因此你需要 fork web-web 和 clash-dashboard

### 部署

在 netlify 上部署静态网站非常简单，只要点击几次鼠标就可以了。

![img](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-35.png)



​	[ 		![img](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-36.png) 	](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-36.png) 

​	[ 		![img](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-37.png) 	](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-37.png) 



![img](https://hsingko.github.io/p/在-linux-上使用-clash-订阅/images/screenshot-38.png)

等待一段时间后，你就可以访问这些网站了。这样做的好处是以后你只需要在本地启动一个 subconverter 就可以了。

## 待解决的问题

目前 subconverter 似乎并不能自动将配置写入到 clash ，需要手动复制，如果遇到订阅内容频繁变更的情况就很麻烦了。

clash 的 api 似乎支持写入配置，我想以后可以写一个脚本自动化地做这件事情。