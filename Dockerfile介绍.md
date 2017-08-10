
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

#### FROM

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
    
    如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写
的指令将作为镜像第一层开始存在。

- 在 Docker 仓库中存在很多高质量的官方镜像
    
    - 官方镜像仓库：[https://store.docker.com](https://store.docker.com)
    - DaoCloud 镜像仓库：[http://hub.daocloud.io](http://hub.daocloud.io)
    - 阿里云镜像仓库：[https://dev.aliyun.com/search.html](https://dev.aliyun.com/search.html)

#### MAINTAINER

> 指定维护者信息。

格式：`MAINTAINER <name>`

#### RUN

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

**提示：在编写 Dockerfile 的时候，要经常提醒自
己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。**

#### CMD

> 指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。

格式：

- `CMD ["executable","param1","param2"]` 使用 exec 执行，推荐方式
- `CMD command param1 param2` 在 /bin/sh 中执行，提供给需要交互的应用
- `CMD ["param1","param2"]` 提供给 ENTRYPOINT 的默认参数

**注意**：

> 1. 如果指定了多条命令，只有最后一条会被执行。
> 2. 如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

#### EXPOSE

> 告诉 Docker 服务端容器暴露的端口号，供互联系统使用。

格式：`EXPOSE <port> [<port>...]`

在启动容器时添加 **-P** 选项，Docker 主机会自动分配一个端口转发到指定的端口。或使用 **-p 80:8080 -p 22:2222 ...** 手动指定

#### ENV

> 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。

格式：`ENV <key> <value>`


#### ADD

> 该命令将复制指定的 <src> 到容器中的 <dest>。 
>
> 其中 <src> 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

格式：`ADD <src> <dest>`


#### COPY

> 复制本地主机的 <src>（为 Dockerfile 所在目录的相对路径）到容器中的 <dest>。

格式：`COPY <src> <dest>`


#### ENTRYPOINT

> 配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。

格式：

- `ENTRYPOINT ["executable", "param1", "param2"]`
- `ENTRYPOINT command param1 param2`

**注意**：每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

#### VOLUME

> 创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

格式：`VOLUME ["/data"]`


#### USER

> 指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。

格式：`USER daemon`

当服务不需要管理员权限时，可以通过该命令指定运行用户，并且可以在之前创建所需要的用户，例如：`RUN groupadd -r postgres && useradd -r -g postgres postgres`。

要临时获取管理员权限可以使用 gosu，而不推荐 sudo。

#### WORKDIR

> 为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。

格式：`WORKDIR /path/to/workdir`

我们可以使用多个 WORKDIR 指令，但是后续命令的参数如果是相对路径，则会基于之前命令指定的路径。例如

```bash
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

最后的路径是 `/a/b/c`

#### ONBUILD

> 配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

格式：`ONBUILD [INSTRUCTION]`



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