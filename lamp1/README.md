lamp1（基于 虚拟机 形式的 Fat Container）
---

> 
>  单一进程容器，又被称为 Fat Container ，类似虚拟机，不推荐
> 


### 1.获取镜像，启动容器

这里使用官方镜像 Ubuntu 16.04，采用加速器 daocloud.io 以解决国内访问 docker-hub 慢的问题，也可以采用阿里云镜像加速器

```
$ docker pull ubuntu:16.04
$ docker run -it ubuntu:16.04
root@86ea8d701da0:/#
```

### 2. 在容器中配置所需要的开发环境

2.1. 替换阿里云软件更新源

```
root@86ea8d701da0:/# apt-get update
root@86ea8d701da0:/# apt-get install -y vim
root@86ea8d701da0:/# vim /etc/apt/sources.list
 
# 16.04
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse

root@86ea8d701da0:/# apt-get update
```

2.2. 安装常用软件包

```
root@86ea8d701da0:/# apt-get install -y net-tools curl wget gcc sudo lsof software-properties-common
```


2.3. 安装 AMP 软件

```
# 安装 MySQL（过程中需要输入数据库的 root 密码）
root@86ea8d701da0:/# apt-get install -y mysql-server mysql-client
root@86ea8d701da0:/# usermod -d /var/lib/mysql/ mysql

# 安装 Apache2
root@86ea8d701da0:/# apt-get install -y apache2
root@86ea8d701da0:/# echo "ServerName localhost" >> /etc/apache2/apache2.conf

# 安装 PHP5.6
root@86ea8d701da0:/# add-apt-repository ppa:ondrej/php
root@86ea8d701da0:/# apt-get update
root@86ea8d701da0:/# apt-get install -y php5.6 libapache2-mod-php5.6 php5.6-dev php5.6-mysql php5.6-gd php5.6-curl php5.6-mbstring --allow-unauthenticated
root@86ea8d701da0:/# pecl install pdo_mysql

# 搜索其它 PHP 可安装插件
root@86ea8d701da0:/# apt-cache search 'php5.6'
```

2.4. 检查服务是否成功启动

```
# 使用 ps -ef 或 netstat -tlunp 查看 MySQL、Apache
# 如果服务没有自动启动，可使用如下命令手动启动
root@86ea8d701da0:/# service mysql start
root@86ea8d701da0:/# service apache2 start
```
2.5. 验证

```
root@86ea8d701da0:/# curl localhost
```

### 3. 清理垃圾

```
root@86ea8d701da0:/# apt-get clean 
root@86ea8d701da0:/# apt-get autoclean 
root@86ea8d701da0:/# rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/log/apache2/* /var/log/mysql/*
root@86ea8d701da0:/# echo /dev/null > ~/.bash_history
```

### 4. 提交变更

```
# 命令基本语法
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

# 查看当前运行中的容器
$ docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
86ea8d701da0        ubuntu:16.04        "/bin/bash"         53 minutes ago      Up 53 minutes                           priceless_clarke

# 提交已经修改的容器为新的镜像
$ docker commit --author "whoru.S.Q <whoru.sun@gmail.com>" --message "init" 86e lamp:v1

# 查看镜像列表
$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
lamp                 v1                  fa80dfae1af0        8 minutes ago       875 MB
ubuntu               16.04               0ef2e08ed3fa        8 weeks ago         130 MB
```

### 5. 以新镜像启动容器

```
# 映射为 8888 端口；本地 www 目录
$ docker run -idt -p 8888:80 -v ~/www:/var/www/html lamp:v1
e629b4c8bb3e3c1c3491e9da2f4a230a22e5172bddaaeb0ced8c3b807393ca2c

# 既然容器手动启动一下服务
$ docker exec -it e62 /bin/bash
root@e629b4c8bb3e:/# service apache2 start
root@e629b4c8bb3e:/# exit

```

浏览器访问：localhost:8888
