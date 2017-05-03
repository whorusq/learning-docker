lamp3（基于 Compose 管理的 LAMP 运行环境）
---

> Compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。
>
> Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。
>
> Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。

### 启动

```bash
$ docker-compose up
```

### 测试

浏览器访问 localhost 或 dev.www（/etc/hosts 文件追加 `127.0.0.1 dev.www`）

```bash
# 如果服务正常，可 ctrl+c 关闭，并追加 -d 参数在后台启动运行
$ docker-compose up -d
```

### 查看启动的服务

```bash
$ docker-compose ps
      Name                     Command               State           Ports          
-----------------------------------------------------------------------------------
lamp_mysql_1        docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp 
lamp_php_apache_1   docker-php-entrypoint apac ...   Up      0.0.0.0:80->80/tcp 
```

### 关闭、删除服务

```bash
$ docker-compose stop
$ docker-compose rm
```

### 注意：关于 docker-compose.yml 的一点说明

* build 对应包含 Dockerfile 文件的目录
* image 对应本地镜像的名称

> 1. 如果 build 和 image 同时指定，在 docker-compose up 时会首先基于 Dockerfile 编译，并指定镜像名称为 image 参数指定的。
> 2. 如果只指定 build，则编译后的镜像名称为`根目录+服务名`，如：lamp3_mysql
> 3. 如果只指定 image，启动时则直接基于对应的本地镜像。
