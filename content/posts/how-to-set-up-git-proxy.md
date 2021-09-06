---
title: "如何设置 Git 代理"
date: 2021-09-04T22:11:56+08:00
tags: [ "git", "proxy" ]
draft: false
---

## TL;DR

下面例子以 GitHub 仓库为例。

### SSH

*~/.ssh/config*
```bash
Host github.com
    ## HTTP
    ProxyCommand nc -v -X connect -x [your_proxy_ip]:[your_http_proxy_port] %h %p
    ## SOCKS5
    ProxyCommand nc -v -x [your_proxy_ip]:[your_socks5_proxy_port] %h %p
```

> 值得一提的是，网上很多案例是这样写的（以 HTTP 为例）：
> 
> ```bash
> Host github.com
>     HostName github.com
>     User git
>     ProxyCommand nc -v -X connect -x [your_proxy_ip]:[your_http_proxy_port] %h %p
> ```
> 
> 可见上述多了 `HostName` 和 `User` 两个指令，分别代表指定**主机名称**和**用户名**。
>
> 但在 clone 代码时的命令为：`git clone git@github.com:[user]/[repository].git`。其中 git 为用户名，github.com 为主机名。根据 [手册描述](https://man.archlinux.org/man/ssh_config.5.en#DESCRIPTION)，SSH 遵循命令行优先，所以这里不用再添加 `HostName` 和 `User` 指令。



### HTTP/HTTPS
```bash
## HTTP
$ git config --global http.proxy http://[your_proxy_ip]:[your_http_proxy_port]
$ git config --global https.proxy http://[your_proxy_ip]:[your_http_proxy_port]
## SOCKS5
$ git config --global http.proxy socks5://[your_proxy_ip]:[your_socks5_proxy_port]
$ git config --global https.proxy socks5://[your_proxy_ip]:[your_socks5_proxy_port]
```

---

## 正文

Git 通过 [四种协议](https://git-scm.com/book/en/v2/Git-on-the-Server-The-Protocols) 下载代码：Local Protocol，HTTP/HTTPS protocols，SSH protocol，Git Protocol。

最常用的是 HTTP/HTTPS 和 SSH，下面分别讨论这两者如何设置代理。

> 假设，我现在的代理服务器的 IP 为：127.0.0.1，分别开启了 SOCKS5(1080) 和 HTTP(1081) 两个端口。

### SSH

使用 `git clone git@github.com:[user]/[repository].git` 的方式克隆代码，就是 SSH 方式。

在 ssh_config 中有个选项为 [ProxyCommand](https://man.archlinux.org/man/ssh_config.5.en#ProxyCommand)，通过它可以指定连接到代理服务器的命令。


而连接代理服务器的命令，可以使用官方推荐的 [openbsd-netcat](https://archlinux.org/packages/community/x86_64/openbsd-netcat/) 来进行连接。

#### openbsd-netcat / netcat / nc

> 注：有多个软件名为 `netcat`，注意不要下错了。具体可以访问 [ArchLinux 的仓库](https://archlinux.org/packages/community/x86_64/openbsd-netcat/) 或 [debian 的仓库](https://packages.debian.org/sid/netcat-openbsd) 进行下载。

在 ArchLinux 中，通过以下命令安装 `openbsd-netcat`：
```bash
$ sudo pacman -S openbsd-netcat
```

`openbsd-netcat` 是一个处理 TCP/IP 的瑞士军刀，在命令行中使用关键字 `netcat` 和 `nc` 来调用。

`openbsd-netcat` 使用 `-x proxy_address[:port]` 连接代理服务器；使用 `-X proxy_protocol` 设置代理的协议。

> 注：[查看文档](https://man.openbsd.org/nc#X) 可知，`openbsd-netcat` 默认使用 SOCKS5(-X 5) 进行对代理服务器的链接。如需使用 HTTP 协议，需加上 `-X connect`。

#### SSH proxy

在 SSH 中，编辑 `~/.ssh/config` 来配置代理。

所有的 SSH 链接都走 HTTP 代理：
```bash
Host *
    ProxyCommand nc -v -X connect -x 127.0.0.1:1081 %h %p
```

> 注：这里的 %h，%p 是 ssh_config 中的 [tokens](https://man.archlinux.org/man/ssh_config.5.en#TOKENS)，是对属性的简写：
> - `%h`：远程主机名。
> - `%p`：远程端口。

只连接 GitHub 时使用 HTTP 代理：
```bash
Host github.com
    ProxyCommand nc -v -X connect -x 127.0.0.1:1081 %h %p
```

连接 GitHub 时使用 SOCKS5 代理：
```bash
Host github.com
    ProxyCommand nc -v -x 127.0.0.1:1080 %h %p
```


### HTTP/HTTPS

使用 `git clone https://github.com/[user]/[repository].git` 的方式克隆代码，就是 HTTP/HTTPS 方式。

Git 中有 [http/https].proxy 选项，直接设置即可：

```bash
## HTTP
$ git config --global http.proxy http://127.0.0.1:1081
$ git config --global https.proxy http://127.0.0.1:1081
## SOCKS5
$ git config --global http.proxy socks5://127.0.0.1:1080
$ git config --global https.proxy socks5://127.0.0.1:1080
```

在 Shell 中，还可以通过设置 [http/https]_proxy 来达到代理的目的：

```bash
## HTTP
$ export http_proxy=http://127.0.0.1:1081
$ export https_proxy=http://127.0.0.1:1081
## SOCKS5
$ export http_proxy=socks5://127.0.0.1:1081
$ export https_proxy=socks5://127.0.0.1:1081
```

## Refs:

- [Tutorial: how to use git through a proxy](http://cms-sw.github.io/tutorial-proxy.html)
- [Use Proxy for Git/GitHub](https://gist.github.com/coin8086/7228b177221f6db913933021ac33bb92)
