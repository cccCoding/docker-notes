## Docker Compose

当业务复杂时，使用容器编排工具，可以批量地创建、调度和管理容器，解决规模化容器的部署问题。Docker 有三种常用的编排工具：Docker Compose、Docker Swarm 和 Kubernetes。

### 介绍

Docker Compose 是 Docker 官方的单机多容器管理系统，它本质是一个 Python 脚本，它通过解析用户编写的 yaml 文件，调用 Docker API 实现动态的创建和管理多个容器。

通常用户服务依赖关系复杂的开发和测试环境，通过 Docker Compose 可一键启动，提高开发效率。线上环境要做到多机集群调度，可使用 Docker Swarm 和 Kubernetes。

### 安装

```shell
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 或者
$ sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 修改执行权限
$ sudo chmod +x /usr/local/bin/docker-compose

# 测试
$ docker-compose --version
```

### 使用

**三个步骤：**

* 编写服务 Dockerfile，保证服务能在任何地方运行
* 编写项目 docker-compose.yml
* 运行 docker-compose up，一键启动项目

### 编写 Docker Compose 模板文件

https://docs.docker.com/compose/compose-file/

在使用 Docker Compose 启动容器时， Docker Compose 会默认使用 **docker-compose.yml** 文件，文件的格式为 yaml。

Docker Compose 模板文件一共有三个版本： v1、v2 和 v3。目前最新的版本为 v3。

Docker Compose 文件主要分为三部分： services（服务）、networks（网络） 和 volumes（数据卷）。

* services（服务）：服务定义了容器启动的各项配置，就像我们执行docker run命令时传递的容器启动的参数一样，指定了容器应该如何启动，例如容器的启动参数，容器的镜像和环境变量等。

* networks（网络）：网络定义了容器的网络配置，就像我们执行docker network create命令创建网络配置一样。

* volumes（数据卷）：数据卷定义了容器的卷配置，就像我们执行docker volume create命令创建数据卷一样。

一个典型的 Docker Compose 文件结构如下：

```yaml
version: "3"
services:
  nginx:
    ## ... 省略部分配置
networks:
  frontend:
  backend:
volumes:
  db-data:
```

### Docker Compose 操作命令

参数

```shell
docker-compose
	-f, --file FILE             指定 docker-compose 文件，默认为 docker-compose.yml
	-p, --project-name NAME     指定项目名称，默认使用当前目录名称作为项目名称
	-d							后台启动？
```

命令

```
  up                 创建并且启动服务
  down               停止服务，并且删除相关资源
  build              构建服务
  config             校验和查看 Compose 文件
  create             创建服务
  start              启动服务
  stop               停止服务
  events             实时监控容器的时间信息
  exec               在一个运行的容器中运行指定命令
  help               获取帮助
  images             列出镜像
  kill               杀死容器
  logs               查看容器输出
  pause              暂停容器
  port               打印容器端口所映射出的公共端口
  ps                 列出项目中的容器列表
  pull               拉取服务中的所有镜像
  push               推送服务中的所有镜像
  restart            重启服务
  rm                 删除项目中已经停止的容器
  run                在指定服务上运行一个命令
  scale              设置服务运行的容器个数
  top                限制服务中正在运行中的进程信息
  unpause            恢复暂停的容器
  version            打印版本信息并退出
```

