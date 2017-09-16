<!-- TOC -->

- [介绍](#介绍)
    - [基本概念](#基本概念)
        - [镜像](#镜像)
            - [分成存储](#分成存储)
            - [虚悬镜像 dangling image](#虚悬镜像-dangling-image)
        - [容器](#容器)
        - [仓库](#仓库)
- [命令](#命令)
    - [run](#run)
        - [参数](#参数)
        - [执行run时Docker在后台执行的操作](#执行run时docker在后台执行的操作)
        - [启动容器并执行命令](#启动容器并执行命令)
            - [在容器中安装ping命令](#在容器中安装ping命令)
        - [以交互模式启动](#以交互模式启动)
        - [后台运行](#后台运行)
    - [start](#start)
    - [终止容器](#终止容器)
    - [commit](#commit)
        - [如何构建自己的镜像](#如何构建自己的镜像)
    - [pull](#pull)
    - [images](#images)
        - [过滤器参数](#过滤器参数)
        - [以特定格式显示](#以特定格式显示)
    - [build](#build)
        - [镜像构建上下文](#镜像构建上下文)
        - [从其他地方获取上下文文件](#从其他地方获取上下文文件)
            - [从 Git repo 构建](#从-git-repo-构建)
            - [用给定的tar压缩包构建](#用给定的tar压缩包构建)
            - [从标准输入中获取上下文压缩包](#从标准输入中获取上下文压缩包)
            - [从标准输入中获取Dockerfile](#从标准输入中获取dockerfile)
    - [save & load](#save--load)
    - [rmi 删除镜像](#rmi-删除镜像)
        - [批量删除](#批量删除)
- [操作容器](#操作容器)
    - [启动容器](#启动容器)
    - [容器和主机间拷贝文件](#容器和主机间拷贝文件)
    - [在容器中构建服务并运行服务](#在容器中构建服务并运行服务)
    - [进入容器](#进入容器)
        - [attach](#attach)
        - [exec](#exec)
    - [导出和导入容器](#导出和导入容器)
        - [导出容器](#导出容器)
        - [导入容器](#导入容器)
            - [load 和 import 的区别](#load-和-import-的区别)
    - [删除容器](#删除容器)
        - [清理所有处于终止状态的容器](#清理所有处于终止状态的容器)
- [数据管理](#数据管理)
    - [数据卷](#数据卷)
        - [创建数据卷](#创建数据卷)
            - [挂载主机目录作为数据卷](#挂载主机目录作为数据卷)
            - [挂载主机文件作为数据卷](#挂载主机文件作为数据卷)
        - [删除数据卷](#删除数据卷)
        - [查看数据卷信息](#查看数据卷信息)
    - [数据卷容器](#数据卷容器)
    - [备份和恢复](#备份和恢复)
        - [备份](#备份)
        - [恢复](#恢复)
- [网络](#网络)
    - [映射端口](#映射端口)
    - [查看映射端口配置](#查看映射端口配置)
    - [容器互联](#容器互联)
- [Dockerfile](#dockerfile)
    - [定制 nginx 服务](#定制-nginx-服务)
    - [FROM](#from)
    - [RUN](#run)
    - [COPY 复制文件](#copy-复制文件)
    - [ADD 更高级的复制文件](#add-更高级的复制文件)
    - [CMD 容器启动命令](#cmd-容器启动命令)
    - [ENTRYPOINT 入口点](#entrypoint-入口点)
        - [场景一: 给命令传递参数](#场景一-给命令传递参数)
        - [场景二: 命令执行前做些准备工作](#场景二-命令执行前做些准备工作)
    - [ENV 设置环境变量](#env-设置环境变量)
    - [ARG 构建参数](#arg-构建参数)
    - [VOLUME 定义匿名卷](#volume-定义匿名卷)
    - [EXPOSE 声明端口](#expose-声明端口)
    - [WORKDIR 指定工作目录](#workdir-指定工作目录)
    - [USER 指定当前用户](#user-指定当前用户)
    - [HEALTHCHECK 健康检查](#healthcheck-健康检查)
    - [ONBUILD 为他人做嫁衣裳](#onbuild-为他人做嫁衣裳)
- [项目](#项目)
    - [redis](#redis)
        - [启动redis服务](#启动redis服务)
        - [连接redis服务](#连接redis服务)
        - [使用自己的配置](#使用自己的配置)
    - [mongo](#mongo)
        - [启动实例](#启动实例)
        - [通过其他应用连接](#通过其他应用连接)
        - [使用mongo连接](#使用mongo连接)
        - [数据存储](#数据存储)
    - [bind](#bind)
    - [openresty](#openresty)
    - [influxdb](#influxdb)
- [参考](#参考)

<!-- /TOC -->

# 介绍

* Docker系统分为服务端和客户端, 服务端管理所有容器, 客户端作为服务端的远程控制器.

## 基本概念

### 镜像

操作系统分为内核和用户空间, 内核启动后, 会挂载 root 文件系统为其提供用户空间的支持, Docker镜像(image)就相当于一个root文件系统. 镜像是一个特殊的文件系统, 除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。 **镜像不包含任何动态数据，其内容在构建之后也不会被改变**.

#### 分成存储

文件系统往往是庞大的, 所以 Docker 利用 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 技术设计为分层存储的架构.

镜像构建时, 会一层一层构建, 前一层是后一层的基础, 每一层构建完后就不再发生变化, 后一层的改变只发生在自己那一层. 比如删除前一层的文件这个操作, 实际上前一层的文件并没有删除, 只是在这一层将其标记成删除, 这个文件依然是保存在最终容器中的, 所以, **构建镜像时尽量只保留需要的东西, 额外的在构建结束前就尽量清理掉**.


#### 虚悬镜像 dangling image

在镜像列表中的一种特殊镜像, 仓库名, 标签均为 `<none>`:

```
<none>               <none>              00285df0df87        5 days ago          342 MB
```

这个镜像原本是有名字的 `mongo:3.2`, 在更新版本时 `docker pull mongo:3.2` 名称被转移到了新版本上, 原来的这个就变成 `<none>` 了, `docker build` 时也会出现这种情况.

使用下面的命令专门显示虚悬镜像:

```bash
docker images -f dangling=true
```

可以使用下面的命令删除:

```bash
$ docker rmi $(docker images -q -f dangling=true)
```



### 容器

镜像是静态的定义, 容器是运行时的实体, 容器可以被创建, 启动, 停止, 删除等. 

容器的实质是进程, 但容器进程是运行在自己独立的命名空间中的, 容器可以拥有自己的 root 文件系统, 自己的网络配置, 自己的进程空间, 自己的用户 ID 空间等. 容器内的进程运行在隔离的环境中, 比直接运行在宿主空间中更安全.

容器运行时是以镜像为基础层, 在其上创建一个供容器读写的存储层. 容器存储层在容器消亡时也随之消亡, 存储在其中的信息也就不存在了.

Docker 最佳实践的要求是: **所有的文件写入操作应该使用数据卷, 或者绑定宿主目录**, 这样会跳过容器存储层, 直接在宿主上读写, 性能和稳定性更高.


### 仓库

公开仓库:

* [Docker Hub](https://hub.docker.com/)
* [CoreOS的Quay.io](https://quay.io/repository/)
* [Google Container Registry](https://cloud.google.com/container-registry/), Kubernetes 使用的就是这个服务.

加速服务:

* [阿里云](https://cr.console.aliyun.com/#/accelerator)
* [DaoCloud 加速器](https://www.daocloud.io/mirror#accelerator-doc)
* [灵雀云](http://docs.alauda.cn/feature/accelerator.html)




# 命令

* `docker version` 查看版本
* `docker images` 显示本地已有的镜像。
* `docker pull ubuntu:14.04` 拉取镜像（保存在 `/var/lib/docker`）
* `docker run -t -i ubuntu:14.04 /bin/bash` 运行镜像并执行命令
* `docker build --rm -t dev:base .` 构建image，`--rm` 删除临时容器, `-t` 指定tag。
* `docker ps` 查看运行中的容器列表.
* `docker inspect TAG`: 查看容器或者镜像的详细信息，除了tag，还可以使用id。
* `docker logs 容器名`: 查看容器的输出, `docker logs -f 容器名`.
* `docker commit -m "message" 容器ID 新镜像名` 提交变动生成新镜像
* `docker search 镜像名字` 在命令行搜索镜像.
* `docker push learn/ping` 发布自己的镜像.
* `docker history 镜像名:标签` 查看镜像内的历史记录.

## run

### 参数

使用 `docker help run` 查看参数

* `-i`: 让容器的标准输入保持打开.
* `-t`: 分配一个伪终端(pseudo-tty), 并绑定到容器的标准输入上.
* `--name`: 指定别名，比如 `docker run --name bob_the_container -i -t ubuntu /bin/bash`。
* `-v`: 挂载目录到容器中，例如：`docker run -it --name testdata -v /home/chen/temp/data:/opt/webapp ubuntu /bin/bash`.
* `--rm`: 容器运行终止后立即删除容器，不能和 `-d` 同时使用。
* `-d`: 后台运行。

### 执行run时Docker在后台执行的操作

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个 ip 地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

### 启动容器并执行命令

```bash
docker run learn/tutorial echo "hello word"
```

#### 在容器中安装ping命令

```bash
docker run learn/tutorial apt-get install -y ping
```

`-y` 参数是避免 `apt-get` 命令进入交互模式.


### 以交互模式启动

```bash
$ docker run -it --rm ubuntu:14.04 bash
root@e7009c6ce357:/# cat /etc/os-release
NAME="Ubuntu"
VERSION="14.04.5 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.5 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
root@e7009c6ce357:/# exit
exit
$
```

### 后台运行

通过添加 `-d` 参数使程序后台运行.

```bash
$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
```

输出结果可以通过 `docker logs [container ID or NAMES]` 查看, 容器信息通过 `docker ps` 查看.



## start

使用 `docker start` 将一个已经终止的容器启动运行.


## 终止容器

* `docker stop` 终止容器.
* 终止的容器可以使用 `docker ps -a` 来查看.
* 终止的容器可以使用 `docker start` 重新启动.
* `docker restart` 是运行中的容器终止,然后启动.



## commit

当对容器做了修改后, 可以通过 `commit` 命令将容器保存为一个新的容器. 和 `git commit` 类似, 它保存新旧状态之间的区别.

1. 使用 `docker ps -l` 查看修改状态后的容器ID, 比如前3位是 698.
2. 执行 `docker commit 698 learn/ping` 保存成新容器 `learn/ping`.

语法: `docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]`, 可以运行 `docker commit` 来查看该命令的参数列表.

### 如何构建自己的镜像

1. 使用 `docker exec -it webserver bash` 进入 webserver 镜像修改内容.
2. 使用 `docker diff webserver` 查看具体的改动.
3. 使用 `docker commit` 保存成新的镜像:
    ```bash
    $ docker commit \
        --author "Tao Wang <twang2218@gmail.com>" \
        --message "修改了默认网页" \
        webserver \
        nginx:v2
    sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214
    ```
4. 使用 `docker history nginx:v2` 查看历史记录.

**慎用 docker commit**

* 通过上面的方式会产生大量的变动, 如果不小心清理, 将使镜像极为臃肿.
* 这种方式生成的镜像称为黑箱镜像, 别人不知道都做了哪些操作.



## pull

```
docker pull [选项] [Docker Registry地址]<仓库名>:<标签>
```

* 选项可以通过 `docker pull --help` 命令看到.
* Docker Registry地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub。
* 仓库名: 两段式名称, `<用户名>/<软件名>`, 用户名默认是 `library`, 官方镜像.


示例:

```bash
$ docker pull ubuntu:14.04
14.04: Pulling from library/ubuntu
bf5d46315322: Pull complete
9f13e0ac480c: Pull complete
e8988b5b3097: Pull complete
40af181810e7: Pull complete
e6f7c7e5c03e: Pull complete
Digest: sha256:147913621d9cdea08853f6ba9116c2e27a3ceffecf3b492983ae97c3d643fbbe
Status: Downloaded newer image for ubuntu:14.04
```

镜像是多层存储构成, 下载也是一层一层下载, 下载中会给出每一层的 ID 的前12位, Digest 给出该镜像的 sha256 摘要.



## images

* 显示顶层镜像: `docker images`
* 显示所有镜像, 包括中间层镜像: `docker images -a`
* 根据仓库名列出镜像: `docker images 仓库名`
    ```bash
    $ docker images ubuntu
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              16.04               f753707788c5        4 weeks ago         127 MB
    ubuntu              latest              f753707788c5        4 weeks ago         127 MB
    ubuntu              14.04               1e0c3dd64ccd        4 weeks ago         188 MB
    ```
* 列出特定的某个镜像:
    ```bash
    $ docker images ubuntu:16.04
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              16.04               f753707788c5        4 weeks ago         127 MB
    ```

### 过滤器参数

`--filter` 或者简写 `-f`

* 过滤出虚悬镜像: `docker images -q -f dangling=true`
* 列举出 `mongo:3.2` 之后建立的镜像: `docker images -f since=mongo:3.2`
* 列举出 `mongo:3.2` 之前建立的镜像: `docker images -f before=mongo:3.2`
* 通过label过滤(如果构建时定义了label): `docker images -f label=com.example.version=0.1`


### 以特定格式显示

只显示镜像ID: `docker images -q`

自定义列, 需要使用 Go 的模板语法, 比如下面只列出 ID 和仓库名:

```bash
$ docker images --format "{{.ID}}: {{.Repository}}"
5f515359c7f8: redis
05a60462f8ba: nginx
fe9198c04d62: mongo
00285df0df87: <none>
f753707788c5: ubuntu
f753707788c5: ubuntu
1e0c3dd64ccd: ubuntu
```

以表格等距显示, 并且有标题行, 和默认一样, 不过自己定义列:

```bash
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY          TAG
5f515359c7f8        redis               latest
05a60462f8ba        nginx               latest
fe9198c04d62        mongo               3.2
00285df0df87        <none>              <none>
f753707788c5        ubuntu              16.04
f753707788c5        ubuntu              latest
1e0c3dd64ccd        ubuntu              14.04
```


## build

格式: `docker build [选项] <上下文路径/URL/->`

示例: `docker build -t nginx:v3 .`

构建多个标签: `docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .`

### 镜像构建上下文

上例中的上下文路径并不是指 Dockerfile 的路径.

Docker 的服务端对外提供一组 REST API, 客户端使用该 API 与 Docker 引擎交互. 在构建过程中, 客户端会将上下文路径指定的目录打包发送给 Docker 引擎, Docker 引擎展开后就得到了构建镜像所需的一切文件. 比如上面的例子中是将当前目录当做上下文目录, 客户端就会将这个目录下的所有内容打包发送给 Docker 引擎.

```bash
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
```

而 `COPY` 这样的命令中的源文件路径就是相对于上下文目录的, 都是相对路径.

如果想将上下文目录中的某些文件省略掉, 可以写一个 `.dockerignore` 文件, 列举出不需要的文件, 语法和 `.gitignore` 一样.

Dockerfile 文件也不是必须存放在上下文路径中的, 甚至可以不叫 Dockerfile, 可以用 `-f ../Dockerfile.php` 来指定.


### 从其他地方获取上下文文件

#### 从 Git repo 构建

```bash
$ docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14
docker build https://github.com/twang2218/gitlab-ce-zh.git\#:8.14
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM gitlab/gitlab-ce:8.14.0-ce.0
8.14.0-ce.0: Pulling from gitlab/gitlab-ce
aed15891ba52: Already exists
773ae8583d14: Already exists
...
```

指定URL地址, 并指定默认的 master 分支, 构建目录是 `/8.14/`(这是repo中的目录???). Docker 会自己去 clone 这个项目, 切换到指定分支, 并进入指定目录开始构建.

#### 用给定的tar压缩包构建

```bash
$ docker build http://server/context.tar.gz
```

Docker会下载这个压缩包, 解压, 以其作为上下文开始构建.


#### 从标准输入中获取上下文压缩包

```bash
$ docker build - < context.tar.gz
```

如果发现输入的文件格式是 gzip, bzip2, xz 的话, 会解压然后将其作为上下文开始构建.

#### 从标准输入中获取Dockerfile

`docker build - < Dockerfile` 或者 `cat Dockerfile | docker build -`

如果传入的是文本文件, 则将其视为 Dockerfile 并开始构建. **这种形式没有上下文, 不可以将本地文件 COPY 进镜像**.




## save & load

将镜像保存为一个 tar 文件，然后传输到另一个位置上，再加载进来。这是在没有 Docker Registry 时的做法，现在已经不推荐，镜像迁移应该直接使用 Docker Registry，无论是直接使用 Docker Hub 还是使用内网私有 Registry 都可以.

比如要保存 alpine 镜像: `docker save alpine | gzip > alpine-latest.tar.gz`

加载镜像: `docker load -i alpine-latest.tar.gz`

如果我们结合这两个命令以及 ssh 甚至 pv 的话，利用 Linux 强大的管道，我们可以写一个命令完成从一个机器将镜像迁移到另一个机器，并且带进度条的功能：

```
docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
```


## rmi 删除镜像

```
docker rmi [选项] <镜像1> [<镜像2> ...]
```

其中, `<镜像>` 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要.

**注意: docker rm 是删除容器, rmi 是删除镜像**

### 批量删除

删除所有虚悬镜像: `docker rmi $(docker images -q -f dangling=true)`

删除所有仓库名为 redis 的镜像: `docker rmi $(docker images -q redis)`

删除所有在 mongo:3.2 之前的镜像: `docker rmi $(docker images -q -f before=mongo:3.2)`





# 操作容器

## 启动容器

1. 方式1: 执行命令然后终止容器 `sudo docker run ubuntu:14.04 /bin/echo 'Hello world'`
2. 方式2: 以交互模式执行 `sudo docker run -t -i ubuntu:14.04 /bin/bash`


## 容器和主机间拷贝文件

1）从容器到主机

```bash
docker cp <containerId>:/file/path/within/container /host/path/target
```

2）从主机到容器

**上面那个命令倒过来就可以了**： `docker cp /host/path/target <containerId>:/file/path/within/container`

Get container name or short container id : `docker ps`

Get full container id： `docker inspect -f '{{.Id}}' SHORT_CONTAINER_ID-or-CONTAINER_NAME`

copy file : `sudo cp path-file-host /var/lib/docker/aufs/mnt/FULL_CONTAINER_ID/PATH-NEW-FILE`


## 在容器中构建服务并运行服务

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




## 进入容器

在使用 -d 参数时，容器启动后会进入后台。 某些时候需要进入容器进行操作.


### attach

```bash
docker attach 容器名
```

**注意: 如果exit，那么容器也会退出。**

示例:

```shell
$ sudo docker run -idt ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
$sudo docker attach nostalgic_hypatia
root@243c32535da7:/#
```

但是使用 attach 命令有时候并不方便。当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了。



### exec

```bash
docker exec -it 容器名 要执行的命令
```

`-it` 是为了获取输入和输出。

比如, 下面进入 webserver 容器修改其中的文件的内容.

```bash
$ docker exec -it webserver bash
root@3729b97e8226:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
root@3729b97e8226:/# exit
exit
```


## 导出和导入容器

### 导出容器

```shell
$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:14.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ sudo docker export 7691a814370e > ubuntu.tar
```

### 导入容器

```shell
$ cat ubuntu.tar | sudo docker import - test/ubuntu:v1.0
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB
```

此外，也可以通过指定 URL 或者某个目录来导入，例如:

```shell
$sudo docker import http://example.com/exampleimage.tgz example/imagerepo
```

#### load 和 import 的区别

用户既可以使用 docker load 来导入镜像存储文件到本地镜像库，也可以使用 docker import 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息.


## 删除容器

可以使用 `docker rm` 来删除一个处于终止状态的容器:

```shell
$sudo docker rm  trusting_newton
trusting_newton
```

如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 SIGKILL 信号给容器。

### 清理所有处于终止状态的容器

```
docker rm $(docker ps -a -q)
```

**注意：这个命令其实会试图删除所有的包括还在运行中的容器，不过 docker rm 默认并不会删除运行中的容器。**



# 数据管理

## 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：
* 数据卷可以在容器之间共享和重用
* 对数据卷的修改会立马生效
* 对数据卷的更新，不会影响镜像
* 数据卷默认会一直存在，即使容器被删除

**注意：数据卷的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的数据卷。**

### 创建数据卷

```shell
$ sudo docker run -d -P --name web -v /webapp training/webapp python app.py
```

* 使用 `-v /webapp` 创建一个数据卷并挂载到**容器的/webapp目录**.
* 可以多次使用 `-v` 挂载多个数据卷.
* 在 Dockerfile 中使用 `VOLUME` 添加数据卷.

#### 挂载主机目录作为数据卷

```shell
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```

* 将主机的 `/src/webapp` 目录挂载到容器的 `/opt/webapp` 目录.
* 本地目录的路径必须是**绝对路径**, 不存在的话会自动创建.
* Dockerfile 中不支持这种做法, 因为 Dockerfile 是为了移植和分享.

Docker 挂载数据卷的默认权限是读写, 可以使用 `:ro` 指定为只读:

```shell
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp:ro
training/webapp python app.py
```

#### 挂载主机文件作为数据卷

```shell
$ sudo docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash
```

这样就可以记录在容器中输入的命令了.

**注意：如果直接挂载一个文件，很多文件编辑工具，包括 vi 或者 sed --in-place，可能会造成文件 inode 的改变，从 Docker 1.1 .0起，这会导致报错误信息。所以最简单的办法就直接挂载文件的父目录。**





### 删除数据卷

* 删除容器的同时移除数据卷: `docker rm -v`


### 查看数据卷信息

```shell
docker inspect web
```

查看其中 `Volumes` 和 `VolumesRW` 的部分. 数据卷都是创建在主机的 `/var/lib/docker/volumes/` 目录下.

Docker1.8.0 起, 数据卷配置在 `Mounts` 下, 创建在主机的 `/mnt/sda1/var/lib/docker/volumes/....` 下.



## 数据卷容器

其实就是一个正常的容器, 只是专门用来提供数据卷供其他容器挂载的, 使得数据可以在容器直接共享.

创建名为 `dbdata` 的数据卷容器:

```shell
$ sudo docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres
```

其他容器通过 `--volumes-from` 来挂载 dbdata 容器中的数据卷:

```shell
$ sudo docker run -d --volumes-from dbdata --name db1 training/postgres
$ sudo docker run -d --volumes-from dbdata --name db2 training/postgres
```

可以使用多个 `--volumes-from` 从多个容器挂载不同的数据卷, 也可以从其他已经挂载了数据卷的容器来级联挂载数据卷:

```shell
$ sudo docker run -d --name db3 --volumes-from db1 training/postgres
```

**使用 --volumes-from 参数所挂载数据卷的容器自己并不需要保持在运行状态**(dbdata不需要运行起来)

如果删除了挂载的容器（包括 dbdata、db1 和 db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时使用 docker rm -v 命令来指定同时删除关联的容器。 这可以让用户在容器之间升级和移动数据卷



## 备份和恢复

### 备份

```shell
$ sudo docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```

先是挂载了 dbdata 容器, 然后将当前目录挂载到这个容器的 `/backup` 目录, 执行 `tar` 目录, 将挂载的 `/dbdata` 目录压缩到 `/backup` 目录, 在本机的当前目录就可以看到这个压缩文件了.

### 恢复

先创建一个带有空数据卷的容器 `dbdata2`:

```shell
$ sudo docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```

再创建一个容器, 挂载 `dbdata2`, 并将当前目录挂载到容器的 `/backup` 目录, 解压文件到 `dbdata2` 的 `/dbdata` 目录:

```shell
$ sudo docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf
/backup/backup.tar
```



# 网络

## 映射端口

* 可以通过 `-p` 或者 `-P` 参数指定端口映射.
* `-P` 会随机映射一个 49000 ~ 49900 的端口到内部容器开放的网络端口.
* `-p` 可以指定要映射的端口, 可以多次使用绑定多个端口.
* 使用 `docker ps` 可以查看端口映射.

映射所有接口的端口`hostPort:containerPort`: `sudo docker run -d -p 5000:5000 training/webapp python app.py`

映射指定地址的端口`ip:hostPort:containerPort`: `sudo docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py`

映射指定地址的任意端口`ip::containerPort`: `sudo docker run -d -p 127.0.0.1::5000 training/webapp python app.py`

映射udp端口: `sudo docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py`

## 查看映射端口配置

```shell
docker port nostalgic_morse 5000
```


## 容器互联

* 连接系统根据容器的名称来执行, 所以容器启动时需要使用 `--name` 指定名称.
* `--link` 参数的格式是 `--link name:alias`, name 是要连接的容器的名称, alias 是这个连接的别名.

比如有一个数据库容器: `docker run -d --name db training/postgres`,
然后创建一个新的容器并连接到db容器: `docker run -d -P --name web --link db:db training/webapp python app.py`

Docker 在互联的容器之间创建了一个安全隧道, 而且不用映射它们的端口到宿主主机上.

Docker 通过2种方式为容器公开连接信息:
1. 环境变量. 可以在 web 容器中通过 `env` 命令查看 `DB_` 开头的环境变量(前缀是设置的别名的大写形式).
2. `/etc/hosts` 文件. 形式为 `172.17.0.5 db`, 可以通过别名 `db` 来访问 `ping db`.









# Dockerfile

* Dcokerfile 是一个文本文件, 其中**每一条指令构建一层**.
* 注释使用 `#`
* 命令换行: 在末尾使用 `\`.


## 定制 nginx 服务

在一个空白命令内新建 Dockerfile 文件:

```
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

在该目录下执行: `docker build -t nginx:v3 .`


## FROM

FROM 用来指定使用哪个镜像为基础镜像, **FROM 命令是必须的, 而且必须是第一行**.

Docker 存在一个特殊的镜像: `scratch`, 这是一个虚拟镜像, 并不存在.

对于 Linux 下静态编译的程序来说, 并不需要操作系统提供运行时支持, 所需的一切库都已经在可执行文件中了, 比如 Go 语言编译的程序. 这也是为什么有人认为 Go 特别适合容器微服务架构语言的原因之一.


## RUN

RUN 用来执行命令, 有两种格式:
1. shell 格式: `RUN <命令>`, 就行在命令行中输入的命令一样.
2. exec 格式: `RUN ["可执行文件", "参数1", "参数2"]`.

如果每个命令前都用 `RUN` 来构建一层镜像, 就会产生非常臃肿的镜像, 而且 Union FS 是有最大层数限制的(127层), 所以可以将多个命令写在一个 `RUN` 中:

```bash
FROM debian:jessie

RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

后面几步进行了清理, 还清理了 apt 缓存.


## COPY 复制文件

两种格式, 一种类似于命令行, 一种类似于函数调用:
* COPY <源路径>... <目标路径>
* COPY ["<源路径1>",... "<目标路径>"]

将上下文目录中的 `<源路径>` 指定的文件或目录复制到镜像内 `<目录路径>` 中, 比如:

```
COPY package.json /usr/src/app/
```

`<源路径>` 可以是多个, 还可以是通配符(规则要符合Go的`filepath.Mathc`):

```
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

`<目标路径>` 可以是绝对路径, 也可以是相对路径, 相对于工作目录, 工作目录可以使用 `WORKDIR` 指令指定. 目标路径不存在时会自动创建.

`COPY` 指令会将源文件的各种元数据都保留, 比如读写执行权限, 文件变更时间等.


## ADD 更高级的复制文件

`ADD` 和 `COPY` 的格式和性质一样, 只是在 `COPY` 的基础上添加了一些功能.

如果源路径是一个 URL, `ADD` 会自动下载并放到目标路径中, 权限为 600(如果想改权限的话还需要增加一层RUN进行调整), 如果下载的是压缩包, 还需要额外的一层 RUN 进行解压(这样的话不如直接使用 wget 或者 curl 下载, 解压缩, 然后清理), 所以不推荐使用.

如果源路径为一个 tar 压缩文件, 压缩格式为 gzip, bzip2, xz 的话, `ADD` 指令会自动解压缩到目标路径中.

所有的文件复制均使用 `COPY`, 仅在需要自动解压缩时使用 `ADD`.


## CMD 容器启动命令

`CMD` 用于容器启动时, 指定默认运行的命令.

格式:
* shell格式: `CMD <命令>`
* exec格式: `CMD ["可执行文件", "参数1", "参数2"...]`
* 参数列表格式: `CMD ["参数1", "参数2"...]`, 使用 `ENTRYPOINT` 后, 用 `CMD` 指定具体的参数.

在运行时也可以使用新命令替代默认命令, 比如 ubuntu 的 `CMD` 是 `/bin/bash`, 直接运行 `docker run -it ubuntu` 的话会直接进入 bash, 如果运行 `docker run -it ubuntu cat /etc/os-release` 的话就是使用 cat 命令替代了 bash 命令.

exec格式的命令会被解析成数组, 因此引号一定要是双引号.

shell格式的命令会被包装为 `sh -c` 的参数形式, 比如 `CMD echo $HOME` 会被转成 `CMD ["sh", "-c", "echo $HOME"]`, 这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理.

**容器中的应用都应该以前台执行**


## ENTRYPOINT 入口点

`ENTRYPOINT` 和 `RUN` 格式一样, 分为 exec 格式和 shell 格式.

`ENTRYPOINT` 和 `RUN` 一样, 都是指定容器启动程序和参数, 当指定了 `ENTRYPOINT` 后, `CMD` 就不再是运行的命令了, 而是变成参数传递给 `ENTRYPOINT`, 变成: `<ENTRYPOINT> <CMD>`

`ENTRYPOINT` 在运行时也可以另外指定命令, 需要使用 `docker run` 的参数 `--entrypoint` 来指定.


### 场景一: 给命令传递参数

假设我们需要一个得知自己公网IP的镜像, 使用 `CMD`:

```bash
FROM ubuntu:16.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://ip.cn" ]
```

当使用 `docker build -t myip .` 构建后, 可以这样使用:

```bash
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通
```

但是现在我们想给 curl 命令传递参数 `-i` 来显示 HTTP 头信息, 使用 `docker run myip -i` 就会报错:

```bash
$ docker run myip -i
docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable file not found in $PATH\"\n".
```

这是因为 Docker 使用 `-i` 替换了 curl 命令.

现在看下使用 `ENTRYPOINT` 的做法:

```bash
FROM ubuntu:16.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
```

这时就可以使用 `docker run myip -i` 命令了. 因为 `-i` 替换了 `CMD` 后被当做参数传递给 `ENTRYPOINT` 了.



### 场景二: 命令执行前做些准备工作

启动容器就是启动主进程, 但有时候启动主进程前需要做些准备工作, 比如配置数据库, 用root身份做些准备然后再以用户身份执行主进程等. 这时可以写一个脚本来做准备工作, 用 `ENTRYPOINT` 来执行脚本, 将 `CMD` 当做参数传递给脚本. 比如redis镜像就是这么做的:

```bash
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
```

`RUN` 中创建了 redis 用户.

```bash
#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
    chown -R redis .
    exec su-exec redis "$0" "$@"
fi

exec "$@"
```

如果 `CMD` 是 `redis-server` 的话, 就以 redis 用户身份启动服务器, 否则以 root 身份启动. 比如:

```bash
$ docker run -it redis id
uid=0(root) gid=0(root) groups=0(root)
```



## ENV 设置环境变量

格式:
* `ENV <key> <value>`
* `ENV <key1>=<value1> <key2>=<value2>...`

示例:

```bash
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```

换行使用 `\`, 含有空格的值需要使用双引号.

使用环境变量的示例(node镜像):

```dockerfile
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

下列指令支持环境变量展开:

```
ADD, COPY, ENV, EXPOSE, LABEL, USER, WORKDIR, VOLUME, STOPSIGNAL, ONBUILD
```


## ARG 构建参数

格式: `ARG <参数名>[=<默认值>]`

构建参数和 ENV 的效果一样，都是设置环境变量。所不同的是，ARG 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有值的.

`ARG` 定义的默认值可以在 `docker build` 时使用 `--build-arg <参数名>=<值>` 来覆盖.

1.13 之前的版本要求 `--build-arg` 中的参数必须用 `ARG` 定义过了, 否则报错. 1.13后的版本放开了限制, 只是警告.


## VOLUME 定义匿名卷

格式为：

* `VOLUME ["<路径1>", "<路径2>"...]`
* `VOLUME <路径>`

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中. 为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据.

```dockerfile
VOLUME /data
```

这里的 `/data` 目录就会在运行时自动挂载为匿名卷，任何向 `/data` 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行时可以覆盖这个挂载设置。比如：

```bash
docker run -d -v mydata:/data xxxx
```

在这行命令中，就使用了 mydata 这个命名卷挂载到了 `/data` 这个位置，替代了 Dockerfile 中定义的匿名卷的挂载配置。



## EXPOSE 声明端口

格式为 `EXPOSE <端口1> [<端口2>...]`

声明容器打算使用什么端口提供服务, 这只是声明, 运行时并不会因为这个声明应用就会开启这个端口的服务。 这样有两个好处: 一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 EXPOSE 的端口。

要将 EXPOSE 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。



## WORKDIR 指定工作目录

格式为 `WORKDIR <工作目录路径>`

使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录）, 以后**各层的当前目录就被改为指定的目录**, 如果该目录不存在，WORKDIR 会帮你建立目录。

一个常见的错误如下:

```dockerfile
RUN cd /app
RUN echo "hello" > world.txt
```

在第一层切换目录后, 仅仅改变了内存而已, 并不能影响到第二层, 因此如果需要改变以后各层的工作目录的位置，那么应该使用 WORKDIR 指令.


## USER 指定当前用户

格式: `USER <用户名>`

USER 则是改变之后层的执行 RUN, CMD 以及 ENTRYPOINT 这类命令的身份。这个用户必须是事先建立好的, 否则无法切换:

```dockerfile
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

如果以 root 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程，不要使用 su 或者 sudo，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用 gosu，可以从其项目网站看到进一步的信息：https://github.com/tianon/gosu

```dockerfile
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.7/gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```

## HEALTHCHECK 健康检查

格式：

* `HEALTHCHECK [选项] CMD <命令>`：设置检查容器健康状况的命令
* `HEALTHCHECK NONE`：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

`HEALTHCHECK` 指令是告诉 Docker 应该如何进行判断容器的状态是否正常，这是 Docker 1.12 引入的新指令。

当在一个镜像指定了 HEALTHCHECK 指令后，用其启动容器，初始状态会为 `starting`，在 HEALTHCHECK 指令检查成功后变为 `healthy`，如果连续一定次数失败，则会变为 `unhealthy`。

HEALTHCHECK 支持下列选项：

* `--interval=<间隔>`：两次健康检查的间隔，默认为 30 秒；
* `--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
* `--retries=<次数>`：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。

和 CMD, ENTRYPOINT 一样，HEALTHCHECK 只可以出现一次，如果写了多个，只有最后一个生效。

在 `HEALTHCHECK [选项] CMD` 后面的命令，格式和 ENTRYPOINT 一样，分为 shell 格式，和 exec 格式。命令的返回值决定了该次健康检查的成功与否：0：成功；1：失败；2：保留，不要使用这个值。


比如我们要检查web服务的状态, 可以这样写:

```dockerfile
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```

为了帮助排障，健康检查命令的输出（包括 stdout 以及 stderr）都会被存储于健康状态里，可以用 `docker inspect` 来查看:

```bash
$ docker inspect --format '{{json .State.Health}}' web | python -m json.tool
{
    "FailingStreak": 0,
    "Log": [
        {
            "End": "2016-11-25T14:35:37.940957051Z",
            "ExitCode": 0,
            "Output": "<!DOCTYPE html>\n<html>\n<head>\n<title>Welcome to nginx!</title>\n<style>\n    body {\n        width: 35em;\n        margin: 0 auto;\n        font-family: Tahoma, Verdana, Arial, sans-serif;\n    }\n</style>\n</head>\n<body>\n<h1>Welcome to nginx!</h1>\n<p>If you see this page, the nginx web server is successfully installed and\nworking. Further configuration is required.</p>\n\n<p>For online documentation and support please refer to\n<a href=\"http://nginx.org/\">nginx.org</a>.<br/>\nCommercial support is available at\n<a href=\"http://nginx.com/\">nginx.com</a>.</p>\n\n<p><em>Thank you for using nginx.</em></p>\n</body>\n</html>\n",
            "Start": "2016-11-25T14:35:37.780192565Z"
        }
    ],
    "Status": "healthy"
}
```


## ONBUILD 为他人做嫁衣裳

格式: `ONBUILD <其它指令>`

ONBUILD 是一个特殊的指令，它后面跟的是其它指令，比如 RUN, COPY 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

Dockerfile 中的其它指令都是为了定制当前镜像而准备的，唯有 ONBUILD 是为了帮助别人定制自己而准备的。

比如构建基础镜像 my-node:

```dockerfile
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
```

带 `ONBUILD` 的命令在构建基础镜像时并不会被执行, 然后在基础镜像的基础上构建各自的镜像:

```dockerfile
FROM my-node
```

此时 `ONBUILD` 后的命令才会执行.



# 项目

## redis

### 启动redis服务

```shell
$ docker run --name some-redis -d redis
```

这个镜像使用了 `EXPOSE 6379`, 互联的容器可以直接使用, 绑定端口的话需要加 `-p 6379:6379`.

持久化存储:

```shell
$ docker run --name some-redis -d redis redis-server --appendonly yes
```

数据会存储在 `VOLUME/data`, 可以使用 `--volumes-from some-volume-container` 或者 `-v /docker/host/dir:/data` 映射到主机目录.

### 连接redis服务

从其他应用连接 redis 服务:

```shell
$ docker run --name some-app --link some-redis:redis -d application-that-uses-redis
```

使用 redis-cli:

```shell
$ docker run -it --link some-redis:redis --rm redis redis-cli -h redis -p 6379
```

### 使用自己的配置

创建自己的 Dockerfile 将自己的配置添加到镜像中:

```dockerfile
FROM redis
COPY redis.conf /usr/local/etc/redis/redis.conf
CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
```

或者通过 `-v` 参数将配置映射到容器中, 启动服务时指定该配置:

```shell
$ docker run -v /myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf --name myredis redis redis-server /usr/local/etc/redis/redis.conf
```




## mongo

### 启动实例

```shell
$ docker run --name some-mongo -d mongo
```

暴露的端口为: `EXPOSE 27017`.

### 通过其他应用连接

```shell
$ docker run --name some-app --link some-mongo:mongo -d application-that-uses-mongo
```

### 使用mongo连接

```shell
$ docker run -it --link some-mongo:mongo --rm mongo sh -c 'exec mongo "$MONGO_PORT_27017_TCP_ADDR:$MONGO_PORT_27017_TCP_PORT/test"'
```

### 数据存储

数据存储在容器的 `/data/db` 目录下.



## bind

```shell
docker run --name bind -d --restart=always \
  --publish 53:53/tcp --publish 53:53/udp --publish 10000:10000/tcp \
  --volume /srv/docker/bind:/data \
  sameersbn/bind:9.9.5-20170626
```

* 后台管理在 `https://localhost:10000`, 用户名 `root`, 密码 `password`, 可以在 run 时通过环境变量指定自己的密码: `--env ROOT_PASSWORD=secretpassword`
* 上面配置的是所有接口都可以访问, 还可以指定 `--publish 127.0.0.1:53:53/udp`.
* 使用 `rndc dumpdb` 后, 文件被存放在 `/var/cache/bind/` 下(或者 ` /var/lib/named/var/cache/bind/`).
* 执行 rndc 命令: `docker exec -it bind rndc status`



## openresty

启动: 

```shell
docker run -d --name="nginx" -p 8080:80 -v $PWD/config/nginx.conf:/usr/local/openresty/nginx/conf/nginx.conf:ro -v $PWD/logs:/usr/local/openresty/nginx/logs openresty/openresty:1.9.15.1-trusty
```

nginx.conf:

```
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 80;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, world</p>")
            ';
        }
    }
}
```

stop:

```shell
docker kill nginx && docker rm nginx
```



## influxdb

启动容器:

```shell
$ docker run -p 8086:8086 \
      -v $PWD:/var/lib/influxdb \
      influxdb
```

生成默认配置文件: `$ docker run --rm influxdb influxd config > influxdb.conf`, 现在在 `$PWD` 下可以修改默认配置文件了.

映射配置文件: 

```shell
$ docker run -p 8086:8086 \
      -v $PWD/influxdb.conf:/etc/influxdb/influxdb.conf:ro \
      influxdb -config /etc/influxdb/influxdb.conf
```

后台启动:

```shell
docker run -p 8086:8086 \
	--name influxdb \
	-d \
	-v $PWD/influxdb.conf:/etc/influxdb/influxdb.conf:ro \
	-v $PWD/influxdb:/var/lib/influxdb \
	influxdb -config /etc/influxdb/influxdb.conf
```


调用 influx-cli:
1. 启动容器: `docker run --name=influxdb -d -p 8086:8086 influxdb`
2. 启动 influx client: `docker run --rm --link=influxdb -it influxdb influx -host influxdb -precision rfc3339`








# 参考

* [Docker官网](https://www.docker.com/)
* [Docker中文社区](http://www.docker.org.cn/)
* [Docker从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/), [GitBook](https://www.gitbook.com/book/yeasy/docker_practice/details)
* [Dockerfile官方文档](https://docs.docker.com/engine/reference/builder/)
* [Dockerfile最佳实践文档](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)