lamp2（基于 Dockerfile 的 Fat Container）
---

> 此种方式将镜像的构建过程通过 Dockerfile 记录下来，并通过构建命令，构建最终镜像，便于对镜像的修改和管理。
> 
> 但是此种方式，仍热是 **Fat Container** 形式，产生的镜像文件比较大，当环境较复杂时 Dockerfile 也会很复杂且不便于整体管理，所以还是不推荐生产环境使用。


### 构建

```
$ docker build -t lamp:v2 .
```

### 运行

```
$ docker run -idt -P -v ~/www:/var/www/html lamp:v2
```

* -v 这里将宿主机家目录下的 www 映射到所启动容器 apache 服务下的 web 根目录
* -P 自动映射开放的端口（-p 4701:80 手动指定）

### 测试

浏览器访问 localhost（默认 80 端口）或 localhost:4701 访问 web
