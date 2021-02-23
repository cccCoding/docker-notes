## Docker 容器

### 简介

容器是基于镜像创建的可运行实例。Docker 镜像都是只读的，当容器启动时，实际上是在容器内部创建该文件系统的读写副本。一个新的可写层被加载到镜像的顶部，这一层就是我们通常说的容器层，该层允许修改镜像的整个副本。容器层之下都叫镜像层。

![image.png](https://s0.lgstatic.com/i/image/M00/4C/D1/CgqCHl9YmlSAGgF0AABXUH--rM4624.png)

### 容器生命周期

容器的生命周期是容器可能处于的状态。

1. created：初建状态
2. running：运行状态
3. stopped：停止状态
4. paused：暂停状态
5. deleted：删除状态

![Lark20200923-114857.png](https://s0.lgstatic.com/i/image/M00/55/BF/CgqCHl9qxcuANmQGAADHS_nfwJE810.png)



### docker run 流程

当使用`docker run`创建并启动容器时，Docker 后台执行的流程为：

* Docker 会检查本地是否存在镜像，如果镜像不存在则从 Docker Hub 拉取镜像；
* 使用镜像创建并启动一个容器；
* 分配文件系统，并且在镜像只读层外创建一个读写层；
* 从 Docker IP 池中分配一个 IP 给容器；
* 执行用户的启动命令运行镜像。

## Docker 数据卷

### 为什么容器需要持久化存储

容器按照业务类型，总体可以分为两类：

- 无状态的（数据不需要被持久化）
- 有状态的（数据需要被持久化）

显然，容器更擅长无状态应用。因为未持久化数据的容器根目录的生命周期与容器的生命周期一样，容器文件系统的本质是在镜像层上面创建的读写层，运行中的容器对任何文件的修改都存在于该读写层，当容器被删除时，容器中的读写层也会随之消失。

虽然容器希望所有的业务都尽量保持无状态，这样容器就可以开箱即用，并且可以任意调度，但实际业务总是有各种需要数据持久化的场景，比如 MySQL、Kafka 等有状态的业务。因此为了解决有状态业务的需求，Docker 提出了卷（Volume）的概念。

Volume 是一个存储概念的封装，不仅支持 local 模式，还支持 nfs 等网络存储。

### 什么是卷

卷的本质是文件或者目录，它可以绕过默认的联合文件系统，直接以文件或目录的形式存在于宿主机上。卷的概念不仅解决了数据持久化的问题，还解决了容器间共享数据的问题。使用卷可以将容器内的目录或文件持久化，当容器重启后保证数据不丢失。

### Docker 卷的操作

使用`docker volume`命令可以实现对卷的创建、查看和删除等操作。

#### 创建数据卷

```shell
docker volume create myvolume
```

默认情况下，Docker 创建的数据卷为 local 模式，仅能提供本主机的容器访问。如果想要实现远程访问，需要借助网络存储来实现。Docker 的 local 存储模式并未提供配额管理，因此在生产环境中需要手动维护磁盘存储空间。

除了使用`docker volume create`的方式创建卷，我们还可以在 Docker 启动时使用 -v 的方式指定容器内需要被持久化的路径，Docker 会自动为我们创建卷，并且绑定到容器中：

```shell
docker run -d --name=nginx-volume -v /usr/share/nginx/html nginx
```

以上命令启动了一个 nginx 容器，`-v`参数使得 Docker 自动生成一个卷并且绑定到容器的 /usr/share/nginx/html 目录中。

使用`docker volume ls`命令来查看下主机上的卷：

```shell
$ docker volume ls
DRIVER              VOLUME NAME
local               eaa8a223eb61a2091bf5cd5247c1b28ac287450a086d6eee9632d9d1b9f69171
```

可以看到，Docker 自动为我们创建了一个名称为随机 ID 的卷。

#### 挂载

* **匿名挂载**

-v 容器内路径

```shell
docker run -d -P nginx01 -v /etc/nginx nginx

docker volume ls	# 生成随机id名称的卷
```

* **具名挂载**

-v 卷名:容器内路径

```shell
docker run -d -P nginx02 -v juming-nginx:/etc/nginx nginx

docker volume ls	# 生成名称为juming-nginx的卷
```

所有 docker 容器内的卷，没有指定目录的情况下，都是在`/var/lib/docker/volumes/xxx/_data` 。

相比较匿名挂载，具名挂载可以方便地找到卷，方便使用。

* **指定路径挂载**

> -v 宿主机路径:容器内路径

**读写权限**

```shell
# -v 容器内路径：ro/rw，改变挂载内容的读写权限
# ro readonly，只读，表明这个路径只能通过宿主机来操作，容器内无法操作
# rw readwrite

docker run -d -P nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P nginx02 -v juming-nginx:/etc/nginx:rw nginx
```

#### 查看数据卷

```shell
docker volume ls
```

#### 查看卷详情

```shell
docker volume inspect myvolume
```

通过`docker volume inspect`命令可以看到卷的创建日期、命令、挂载路径信息。

#### 使用数据卷

在容器启动时添加 --mount 参数指定卷的名称，即可使用`docker volume create`创建的卷。

这里我们使用上一步创建的卷来启动一个 nginx 容器，并将 /usr/share/nginx/html 目录与卷关联：

```shell
docker run -d --name=nginx --mount source=myvolume,target=/usr/share/nginx/html nginx
```

使用 Docker 的卷可以实现指定目录的文件持久化，进入容器中并在 /usr/share/nginx/html 目录下新增 index.html 文件内容，将容器删除，启动新的容器并再次挂载 myvolume 卷，文件数据依然在。

#### **删除数据卷**

删除容器并不会自动删除已经创建的数据卷，因此不再使用的数据卷需要我们手动删除:

```shell
docker volume rm myvolume
```

需要注意，正在被使用中的数据卷无法删除，如果你想要删除正在使用中的数据卷，需要先删除所有关联的容器。

#### 容器与容器之间数据共享

创建共享数据卷

```shell
docker volume create log-vol
```

启动一个生产日志的容器（producer）

```shell
docker run --mount source=log-vol,target=/tmp/log --name=log-producer -it busybox
```

启动一个消费者容器（consumer）

```shell
docker run -it --name consumer --volumes-from log-producer busybox
```

使用`volumes-from`参数可以在启动新的容器时来挂载已经存在的容器的卷，`volumes-from`参数后面跟已经启动的容器名称。

从 producer 容器写入的文件内容会自动出现在 consumer 容器中，两个容器间的数据共享。

数据卷的生命周期持续到没有容器使用为止。

B 挂载 A，则称 A 为父容器（数据卷容器），B 为子容器。

#### 主机与容器之间数据共享

Docker 卷的目录默认在 /var/lib/docker 下，如果想把主机的其他目录映射到容器内，则需要用到主机与容器之间数据共享的方式。

要实现主机与容器之间数据共享很简单，只需要我们在启动容器的时候添加`-v`参数即可，使用格式为：`-v HOST_PATH:CONTIANAER_PATH`。

例如，我想挂载主机的 /data 目录到容器中的 /usr/local/data 中，可以使用以下命令来启动容器：

```shell
docker run -v /data:/usr/local/data -it busybox
```

容器启动后，便可以在容器内的 /usr/local/data 访问到主机 /data 目录的内容了，并且容器重启后，/data 目录下的数据也不会丢失。

#### 小结

![Lark20201010-145710.png](https://s0.lgstatic.com/i/image/M00/5C/50/Ciqc1F-BW1SAQEkaAACOwJuMTHI950.png)

### Docker 卷的实现原理

先回顾一下镜像和容器的文件系统原理： 镜像是由多层文件系统组成的，当我们想要启动一个容器时，Docker 会在镜像上层创建一个可读写层，容器中的文件都工作在这个读写层中，当容器删除时，与容器相关的工作文件将全部丢失。

Docker 容器的文件系统不是一个真正的文件系统，而是通过联合文件系统实现的一个伪文件系统，而 Docker 卷则是直接利用主机的某个文件或者目录，它可以绕过联合文件系统，直接挂载主机上的文件或目录到容器中，这就是它的工作原理。

**Docker 卷的实现原理，即在我们创建 Docker 卷时，在主机的 /var/lib/docker/volumes 目录下，根据卷的名称创建相应的目录，然后在每个卷的目录下创建 _data 目录。在容器启动时如果使用 --mount 参数，Docker 会把主机上的 _data 目录映射到容器的指定目录下，实现数据持久化。因此我们在容器中挂载卷的目录下操作文件，实际上是在操作主机上的 _data 目录。**

如何使用卷来挂载 NFS 类型的持久化存储到容器内？

### Dockerfile 中的卷

创建 Dockerfile

```shell
# 指令都大写
FROM centos

# 生成两个匿名挂载卷
VOLUME ["volume01","volume02"]

CMD echo "----end----"

CMD /bin/bash
```

构建镜像

```shell
docker build -t mycentos:1.0 .
```

