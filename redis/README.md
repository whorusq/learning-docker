<p align="center">
	<a href="https://redis.io">
		<img src="./redis_logo.png" attr="rsync logo" title="官网：https://redis.io" width="60%">
	</a>
</p>


### 1. 相关资源

- 官网：https://redis.io
- 镜像地址：https://store.docker.com/images/redis
- redis 发布地址：https://github.com/antirez/redis

### 2. 拉取镜像

```bash
# 默认拉取最新稳定版本，目前是 5.x
➜  docker pull redis

# 或指定版本
➜  docker pull redis:5
```
检查镜像是否拉取成功

```bash
➜  docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               5                   4cdbec704e47        9 days ago          98.2MB
redis               latest              44d36d2c2374        2 months ago        98.2MB
```

### 3. 使用

#### 3.1. 默认启动

```bash
➜  docker run --name myredis -d redis
41f9c2e8fb86c415257c0342eb58435c3a5bbaf10f888a2cb3c7243d08ec796c
```
之后就可以通过客户端程序连接 `127.0.0.1:6379` 来访问了。

#### 3.2. 怎么使用 redis-cli 连接容器？

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

#### 3.3. 如何将数据持久化存储到宿主机？

```bash
➜  docker run --name myredis2 -d -v ~/data/redis:/data redis redis-server --appendonly yes
```
说明：

- `--appendonly yes` 用于打开 redis 的数据持久化存储
- `-v ~/data/redis:/data` 用于将宿主机的目录映射到容器对应的数据存储目录

#### 3.4. 自定义配置文件

建议[官网](https://redis.io)下载指定版本的 Redis 源码包，解压后获取默认配置文件 `redis.conf`

然后在 Redis 容器启动时如下操作：

```bash
# ~/myredis/conf/redis.conf 对应宿主机配置文件位置
➜  docker run -v ~/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf --name myredis3 redis redis-server /usr/local/etc/redis/redis.conf
```

### 4. 使用 docker compose 编排

上面是基本使用方式，为便于管理，我们一般使用 `docker compose` 来统一创建、管理各服务。

#### 4.1. 创建

```bash
# 新建一个目录 redi 存放，结构如下
➜  tree -L 2 redis
.
├── conf
│   └── redis.conf <-- 默认配置文件
├── data
│   └── dump.rdb <-- 数据持久化文件
└── docker-compose.yml
```

`docker-compose.yml` 文件如下：

```yml
# Redis
# https://redis.io
# https://hub.docker.com/_/redis
version: '3.1'

services:

  redis:
    image: redis:5
    restart: always
    ports:
      - "56379:6379"
    volumes:
      - ./data:/data:rw
      - ./conf/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
```

#### 4.2. 启动

> ⚠️ 注意：启动前，注意修改配置文件中的连接密码（requirepass）和 IP 绑定（bind）两个属性，尤其是当使用 `redis-cli` 或其它客户端工具连接发生异常时。

```bash
➜  docker compose up
Creating network "redis_default" with the default driver
Creating redis_redis_1 ... done
Attaching to redis_redis_1
redis_1  | 1:C 10 Apr 2020 07:24:54.884 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 10 Apr 2020 07:24:54.885 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 10 Apr 2020 07:24:54.885 # Configuration loaded
redis_1  |                 _._
redis_1  |            _.-``__ ''-._
redis_1  |       _.-``    `.  `_.  ''-._           Redis 5.0.8 (00000000/0) 64 bit
redis_1  |   .-`` .-```.  ```\/    _.,_ ''-._
redis_1  |  (    '      ,       .-`  | `,    )     Running in standalone mode
redis_1  |  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
redis_1  |  |    `-._   `._    /     _.-'    |     PID: 1
redis_1  |   `-._    `-._  `-./  _.-'    _.-'
redis_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|
redis_1  |  |    `-._`-._        _.-'_.-'    |           http://redis.io
redis_1  |   `-._    `-._`-.__.-'_.-'    _.-'
redis_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|
redis_1  |  |    `-._`-._        _.-'_.-'    |
redis_1  |   `-._    `-._`-.__.-'_.-'    _.-'
redis_1  |       `-._    `-.__.-'    _.-'
redis_1  |           `-._        _.-'
redis_1  |               `-.__.-'
redis_1  |
redis_1  | 1:M 10 Apr 2020 07:24:54.890 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_1  | 1:M 10 Apr 2020 07:24:54.891 # Server initialized
redis_1  | 1:M 10 Apr 2020 07:24:54.891 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
redis_1  | 1:M 10 Apr 2020 07:24:54.901 * DB loaded from disk: 0.009 seconds
redis_1  | 1:M 10 Apr 2020 07:24:54.901 * Ready to accept connections
```

#### 4.3. 连接测试

```bash
➜  docker exec -it ${Redis 容器 ID} redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379>
```
