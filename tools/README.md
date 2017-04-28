### 镜像描述

+ 基于官方镜像 Ubuntu 16.04
+ 安装常用工具 

	- software-properties-common
	- wget
	- curl
	- vim
	- ssh
	- sudo
	- supervisor
	- lsof
	- zip
	- unzip
	- net-tools
	- inetutils-ping

+ 添加普通用户 admin/admin，可以 ssh 登录
+ 使用 supervisor 启动、管理进程

### 编译镜像

```bash
$ docker build -t tools:0.2 .
```

### 启动容器

```
$ docker run -idt -p 4701:22 tools:0.2
```