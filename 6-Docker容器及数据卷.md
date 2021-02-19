## Docker 容器

### 简介

容器是基于镜像创建的可运行实例。运行容器化环境时，实际上实在容器内部创建该文件系统的读写副本。这将添加一个容器层，该层允许修改镜像的整个副本。

![image.png](C:\Users\Administrator\Desktop\容器\docker-notes\6-容器数据卷.assets\CgqCHl9YmlSAGgF0AABXUH--rM4624.png)

### 容器生命周期

容器的生命周期是容器可能处于的状态。

1. created：初建状态
2. running：运行状态
3. stopped：停止状态
4. paused：暂停状态
5. deleted：删除状态

![Lark20200923-114857.png](https://s0.lgstatic.com/i/image/M00/55/BF/CgqCHl9qxcuANmQGAADHS_nfwJE810.png)



当使用docker run创建并启动容器时，Docker 后台执行的流程为：

* Docker 会检查本地是否存在 busybox 镜像，如果镜像不存在则从 Docker Hub 拉取 busybox 镜像；
* 使用 busybox 镜像创建并启动一个容器；
* 分配文件系统，并且在镜像只读层外创建一个读写层；
* 从 Docker IP 池中分配一个 IP 给容器；
* 执行用户的启动命令运行镜像。

### 容器数据卷

#### 什么是容器数据卷

容器之间可以数据共享的技术。数据持久化。Docker容器中产生的数据同步到本地。

目录挂载，将容器内的目录挂载到容器外。

避免删除容器时数据一起丢失。

#### 使用后数据卷

> 方式一：使用命令 -v

```shell
docker run -d -v 主机目录:容器内目录 mysql:5.7
```

> 方式二：



#### 具名和匿名挂载

**匿名挂载**

> -v 容器内路径

```shell
docker run -d -P nginx01 -v /etc/nginx nginx

docker volume ls
```

**具名挂载**

> -v 卷名:容器内路径

```shell
docker run -d -P nginx02 -v juming-nginx:/etc/nginx nginx

docker volume ls

docker volume inspect juming-nginx
```

所有docker容器内的卷，没有指定目录的情况下，都是在`/var/lib/docker/volumes/xxx/_data` 。

通过具名挂载可以方便地找到卷，推荐食用。

**指定路径挂载**

> -v /宿主机路径:容器内路径

**扩展**

```shell
# -v 容器内路径：ro/rw ，改变读写权限
# ro readonly，只读，表明这个路径只能通过宿主机来操作，容器内无法操作
# rw readwrite

# 一旦设置了容器权限，容器对我们挂载出来的内容就有限定了
docker run -d -P nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P nginx02 -v juming-nginx:/etc/nginx:rw nginx
```

#### 初识Dockerfile

Dockerfile就是用来构建docker镜像的文件

创建 dockerfile01

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
docker build -f dockerfile01 -t mycentos:1.0 .
```

#### 数据卷容器

多个mysql同步数据

> --volumes-from

B挂载A，则A为父容器（数据卷容器），B为子容器

```shell
docker run -it --name docker01 centos

docker run -it --name docker02 --volumes-from docker01

docker run -it --name docker03 --volumes-from docker01
```

可是实现容器之间配置共享。

数据卷的生命周期持续到没有容器使用为止。

