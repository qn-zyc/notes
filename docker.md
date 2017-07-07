<!-- TOC -->

- [1. 命令列表](#1-命令列表)
    - [1.1. run](#11-run)
        - [1.1.1. 参数](#111-参数)
- [2. 容器和主机间拷贝文件](#2-容器和主机间拷贝文件)
- [3. 在容器中构建服务并运行服务](#3-在容器中构建服务并运行服务)
- [4. 挂载数据卷](#4-挂载数据卷)
- [5. 容器互联](#5-容器互联)
- [6. 进入容器](#6-进入容器)
    - [6.1. attach](#61-attach)
    - [6.2. exec](#62-exec)

<!-- /TOC -->

# 1. 命令列表

* `docker images` 显示本地已有的镜像。
* `docker pull ubuntu:14.04` 拉取镜像（保存在 `/var/lib/docker`）
* `docker run -t -i ubuntu:14.04 /bin/bash` 运行镜像并执行命令
* `docker build --rm -t dev:base .` 构建image，`--rm` 删除临时容器, `-t` 指定tag。
* `docker inspect TAG`: 查看容器或者镜像的信息，除了tag，还可以使用id。
* `docker logs 容器名`: 查看容器的输出
* `docker commit -m "message" 容器ID 新镜像名` 提交变动生成新镜像

## 1.1. run

### 1.1.1. 参数

使用 `docker help run` 查看参数

`-i`: 保证容器中STDIN是开启的， 尽管我们并没有附着到容器中。 持久的标准输入是交互式shell的“半边天”.
`-t`: 它告诉Docker为要创建的容器分配一个伪tty终端。 这样，新创建的容器才能提供一个交互式shell。
`--name`: 指定别名，比如 `docker run --name bob_the_container -i -t ubuntu /bin/bash`。
`-t`: 挂载目录到容器中，例如：`docker run -it --name testdata -v /home/chen/temp/data:/opt/webapp ubuntu /bin/bash`.
`--rm`: 容器运行终止后立即删除容器，不能和 `-d` 同时使用。
`-d`: 后台运行。



# 2. 容器和主机间拷贝文件

1）从容器到主机

```bash
docker cp <containerId>:/file/path/within/container /host/path/target
```

2）从主机到容器

**上面那个命令倒过来就可以了**： `docker cp /host/path/target <containerId>:/file/path/within/container`

Get container name or short container id : `docker ps`

Get full container id： `docker inspect -f '{{.Id}}' SHORT_CONTAINER_ID-or-CONTAINER_NAME`

copy file : `sudo cp path-file-host /var/lib/docker/aufs/mnt/FULL_CONTAINER_ID/PATH-NEW-FILE`


# 3. 在容器中构建服务并运行服务

Dockerfile:

```
FROM ubuntu
MAINTAINER Docker Chen <zhangyuchen@wepiao.com>

# 复制文件
ADD test_server /home/server
# 暴露容器的8080端口
EXPOSE 8080

# 运行程序
ENTRYPOINT ["/home/server"]
```

构建镜像：

```
docker build -t="chen/testserver:v1" .
```

启动容器：

```
docker run -d -p 8080:8080 chen/testserver:v1
```

`-d` 后台运行, `-p` 指定容器的8080端口和主机的8080端口互通


# 4. 挂载数据卷

```
docker run -it -v /home/chen/dkdata:/mnt/dbdata --name dbdata ubuntu /bin/bash
```

挂载主机的 `/home/chen/dkdata` 到容器的 `/mnt/dbdata`， 可以在容器中访问到主机的文件。

```
docker run -it --volumes-from dbdata --name db1 ubuntu
```

db1容器挂载了dbdata容器，这样在db1容器的 `/mnt/dbdata` 中就可以访问挂载的数据了。


# 5. 容器互联

先启动一个测试服务：`docker run -it --rm --name testserver chen/testserver:v1`， 监听8080端口， 这个容器没有使用`-p`绑定主机端口， 所以从外部访问不了里面的服务。

再启动一个容器`runcurl`连接`testserver`容器： `docker run -it --name runcurl --link testserver:server chen/base /bin/bash`

`--link` 的格式是 `name:alias`。

测试一下是否连通了： `curl server:8080/a/b`

server对应的IP可以通过 `/etc/hosts` 文件查看。


# 6. 进入容器

## 6.1. attach

```bash
docker attach 容器名
```

**注意: 如果exit，那么容器也会退出。**

## 6.2. exec

```bash
docker exec -it 容器名 要执行的命令
```

`-it` 是为了获取输入和输出。

