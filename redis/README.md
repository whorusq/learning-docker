
### 1. 相关资源

- 镜像地址：https://store.docker.com/images/redis
- redis 发布地址：https://github.com/antirez/redis

拉取镜像

```bash
# 默认拉取最新版本，目前是 4.0.1
➜  docker pull redis

# 或指定版本
➜  docker pull redis:3
```
检查镜像是否拉取成功

```bash
➜  docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              d4f259423416        5 weeks ago         106MB
```

### 2. 使用

#### 2.1. 默认启动

```bash
➜  docker run --name myredis -d redis
41f9c2e8fb86c415257c0342eb58435c3a5bbaf10f888a2cb3c7243d08ec796c
```
之后就可以通过客户端程序连接 `127.0.0.1:6379` 来访问了

#### 2.2. 怎么使用 redis-cli 连接容器？

```bash
➜  docker run -it --link myredis --rm redis redis-cli -h myredis -p 6379
myredis:6379> KEYS *
(empty list or set)
myredis:6379> SET name 'xiaoming'
OK
myredis:6379> GET name
"xiaoming"
myredis:6379> exit
```

#### 2.3. 如何将数据持久化存储到宿主机？

```bash
➜  docker run --name myredis2 -d -v ~/data/redis:/data redis redis-server --appendonly yes
```
说明：

- `--appendonly yes` 用于打开 redis 的数据持久化存储
- `-v ~/data/redis:/data` 用于将宿主机的目录映射到容器对应的数据存储目录

#### 2.4. 自定义配置文件

首先到 [https://github.com/antirez/redis/blob/unstable/redis.conf](https://github.com/antirez/redis/blob/unstable/redis.conf) 下载一份 redis 的默认配置文件，然后在 redis 容器启动时如下操作：

```bash
# ~/myredis/conf/redis.conf 对应宿主机配置文件位置
➜  docker run -v ~/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf --name myredis3 redis redis-server /usr/local/etc/redis/redis.conf
```
