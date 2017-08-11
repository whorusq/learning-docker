
Dockerfile介绍
---

### 目录

1. [Dockerfile 基本结构](#1-基本结构)
2. [Dockerfile 常用指令](#2-常用指令)
	- 2.1. [FROM](#21-from)
	- 2.2. [MAINTAINER](#22-MAINTAINER)
	- 2.3. [RUN](#23-RUN)
	- 2.4. [CMD](#24-CMD)
	- 2.5. [ENTRYPOINT](#25-entrypoint)
	- 2.6. [ENV](#26-env)
	- 2.7. [COPY](#27-copy)
	- 2.8. [ADD](#28-add)
	- 2.9. [VOLUME](#29-volume)
	- 2.10. [EXPOSE](#210-expose)
	- 2.11. [USER](#211-user)
	- 2.12. [WORKDIR](#212-workdir)
	- 2.13. [ONBUILD](#213-onbuild)
3. [基于 Dockerfile 构建镜像](#3-构建镜像)

---

### 1. 基本结构

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

在文件中，支持以 # 开头的注释行。

一般，Dockerfile 共包括四部分：

- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行指令

使用流程

```bash
➜ mkdir mynginx
➜ cd mynginx
➜ touch Dockerfile
➜ vi Dockerfile
```

写入如下内容

```
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index
.html
```

以上是一个最简单的 Dockerfile，相对完整版的 Dockerfile 示例，请[点击这里](https://github.com/whorusq/docker-learning/tree/master/lamp2)查看。

### 2. 常用指令

#### 2.1. FROM

> 指定当前 Dockerfile 文件所创建的镜像基于哪个基础镜像

格式：

- `FROM <image>`
- `FROM <image>:<tag>`

**注意**：
- 一个 Dockerfile 中 FROM 是必备的指令，并
且必须是第一条指令。
- Docker 还存在一个特殊的镜像，名为
scratch 。
    
    它是个虚拟概念，并不实际存在，表示一个空白的镜像。
    
    ```
    FROM scratch
    ...
    ```
    
    如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

- 在 Docker 仓库中存在很多高质量的官方镜像
    
    - 官方镜像仓库：[https://store.docker.com](https://store.docker.com)
    - DaoCloud 镜像仓库：[http://hub.daocloud.io](http://hub.daocloud.io)
    - 阿里云镜像仓库：[https://dev.aliyun.com/search.html](https://dev.aliyun.com/search.html)

#### 2.2. MAINTAINER

> 指定维护者信息。

格式：

- `MAINTAINER <name>`

#### 2.3. RUN

> 每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

格式：

- `RUN <command>` 在 shell 终端中运行命令，即 `/bin/sh -c`
- `RUN ["executable", "param1", "param2"]` 使用 exec 执行

比如，我们想指定使用其它终端，就可以通过第二种方式实现，例如 `RUN ["/bin/bash", "-c", "echo hello"]`

**注意**：Dockerfile 中每一个指令都会建立一层， RUN 也不例外。所以在编写 Dockerfile 的过程中，我们应该尽量避免无意义的层次，采取类似如下示例的操作：

```bash
FROM ubuntu:16.04

RUN apt-get update \
	
	# 安装基础工具包
	&& apt-get install -y software-properties-common vim wget  \
    
    # 这里追加其它操作：安装软件、配置服务等
    # ...
    
	# 做一些清理工作
	&& apt-get clean \
    && apt-get autoclean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* 
```

**提示：在编写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。**


#### 2.4. CMD

> 指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。

格式：

- `CMD ["executable","param1","param2"]` 使用 exec 执行，**推荐**方式
    
    > 示例：`CMD ["nginx", "-g", "daemon off;"]`

- `CMD command param1 param2` 在 /bin/sh 中执行，提供给需要交互的应用
- `CMD ["param1","param2"]` 提供给 ENTRYPOINT 的默认参数

Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。 CMD 指令就是用于指定默认的容器主进程的启动命令的。

ubuntu 镜像默认的 CMD 是 `/bin/bash`。

**注意**：

> 1. 如果指定了多条命令，只有最后一条会被执行。
> 2. 如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

#### 2.5. ENTRYPOINT

> 与 CMD 一样，都是配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。

格式：

- `ENTRYPOINT ["executable", "param1", "param2"]`
- `ENTRYPOINT command param1 param2`

当指定了 ENTRYPOINT 后， CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令，换句话说实际执行时，将变为：

```
<ENTRYPOINT> "<CMD>"
```

**注意**：每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。


#### 2.6. ENV

> 设置环境变量，会被后续 RUN 指令使用，并在容器运行时保持。

格式：

- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>...`

示例：

```
ENV VERSION=1.0 DEBUG=on \

    NAME="Happy Feet"
```

#### 2.7. COPY

> 复制本地主机的 <src>（为 Dockerfile 所在目录的相对路径）到容器中的 <dest>。

格式：

- `COPY <源路径>... <目标路径>`
- `COPY ["<源路径1>",... "<目标路径>"]`

**<源路径>** 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 filepath.Match 规则，如：

```
COPY hom* /mydir/

COPY hom?.txt /mydir/
```

**<目标路径>** 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工 作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建 相关文件都在使用 Git 进行管理的时候。

#### 2.8. ADD

> 与 COPY 的格式和性质基本一致，但是在 COPY 基础上增加了一些功能。
>
> 其中 <src> 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

在 Docker 官方的最佳实践文档中要求，尽可能的使用 COPY ，因为 COPY 的语义很明确，就是复制文件而已，而 ADD 则包含了更复杂的功能，其行为也不一定很清晰。**最适合使用 ADD 的场合，就是所提及的需要自动解压缩的场合。**

**注意**：

> ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。
> 
> 因此在 COPY 和 制均使用 COPY 指令中选择的时候，可以遵循这样的原则：**所有的文件复制均使用 COPY 指令，仅在需要自动解压缩的场合使用 ADD 。**

#### 2.9. VOLUME

> 定义匿名卷。
>
> 即创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

格式：

- `VOLUME ["<path1>", "<path2>"...]`
- `VOLUME path`

运行时可以通过 **-v** 选项覆盖这个挂载设置，如：

```
docker run -d -v mydata:/data xxxx
```

#### 2.10. EXPOSE

> 告诉 Docker 服务端容器暴露的端口号，供互联系统使用。
>
> 

格式：

- `EXPOSE <port> [<port>...]`

**注意**：

1. 这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处：

    - 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映 射；
    - 用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 EXPOSE 的端口。

2. 要将 EXPOSE 和在运行时使用 -p <宿主端口>:<容器端口> 区分开来。 

    - -p ，是 映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问；
    - EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。


#### 2.11. USER

> 指定运行容器时的用户名或 UID，后续的 RUN / CMD / ENTRYPOINT 也会使用指定用户。

格式：

- `USER daemon`

与 WORKDIR 指令一样， USER 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。

示例：

```
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

要临时获取管理员权限可以使用 **gosu**，而不推荐 sudo，可参照：[https://github.com/tianon/gosu](https://github.com/tianon/gosu)

使用示例：

```
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis

# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.7/gosu-amd64" \
&& chmod +x /usr/local/bin/gosu \
&& gosu nobody true

# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```

#### 2.12. WORKDIR

> 为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。

格式：

- `WORKDIR /path/to/workdir`

我们可以使用多个 WORKDIR 指令，但是后续命令的参数如果是相对路径，则会基于之前命令指定的路径。例如

```bash
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

最后的路径是 `/a/b/c`

#### 2.13. ONBUILD

> 配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

格式：

- `ONBUILD [INSTRUCTION]`



### 3. 构建镜像

编写完成 Dockerfile 之后，我们就可以通过 docker build 命令来创建镜像了。

格式：`docker build [选项] 路径`

在 Dockerfile 所在目录，执行如下命令进行构建：

```bash
➜ sudo docker build -t php:v5.6 .
➜ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
php                 5.6                 cb4854aef367        3 months ago        351MB
```

> 其中 **-t** 用于指定镜像的标签信息；**\.** 表示构建上下文的目录