## Docker概述

### 概述

Docker 是一个用于开发，发布和运行应用程序的开放平台。是基于Go语言开发的开源的应用容器引擎。可以让开发者打包应用及依赖包到一个轻量级、可移植的容器中，然后发布到Linux服务器上。

从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版），用社区版即可。

文档地址 https://docs.docker.com/

仓库地址 https://hub.docker.com/

### Docker应用场景

1. Web 应用的自动化打包和发布。
2. 自动化测试和持续集成、发布。

### Docker的优点

- 将软件和环境一起打包，快速、一致地交付应用程序
- 适合持续集成和持续交付（CI / CD）工作流程
- 轻量
  - 在同一硬件上运行更多工作负载，提高服务器利用率
  - 容器化技术不是模拟的一个完整的操作系统，因此要轻量，体积小，资源占用小，启动快
- 沙箱机制，每个容器之间相互隔离，每个容器内都有一个属于自己的文件系统，互不影响，更加安全
- 响应式部署和扩展
  - Docker 的可移植性和轻量级特性，能够轻松地完成动态管理（升级和扩缩容）

对比虚拟机：资源占用多、冗余步骤多、启动慢

### Docker的用处

> DevOps （开发、运维）

* 应用更快速地交付和部署
  * 传统：一堆帮助文档，安装程序
  * Docker：打包镜像发布测试，一键运行
* 更便捷地升级和扩缩容
* 更简单地系统运维
  * 开发、测试环境高度一致
* 更高效地计算资源利用
  * Docker是内核级别的虚拟化，可以在一个物理机上运行很多的容器实例，服务器的性能能被压榨到极致。

### Docker 核心概念

Docker 包括以下三个基本概念：

- **镜像（Image）：**相当于一个root文件系统。是一个模板，可以通过这个模板来创建容器。
- **容器（Container）：**镜像和容器的关系，就像是类与对象，镜像是静态的定义，容器是镜像的运行实体。容器可以被创建、启动、停止、删除等。容器运行着真正的应用进程。
- **仓库（Repository）：**类似代码仓库，用来存放和分发镜像。分为公有仓库和私有仓库。

## Docker架构

Docker 采用 C/S （客户端/服务端）架构。客户端负责发送操作指令，服务端负责接收和处理指令。客户端和服务端通信有多种方式，既可以在同一台机器上通过UNIX套接字通信，也可以通过网络连接远程通信。

### **架构图**

![Drawing 2.png](https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYtCAC1GSAADIK4E6wrc368.png)

### Docker 客户端

Docker 客户端其实是一种泛称。其中 docker 命令是 Docker 用户与 Docker 服务端交互的主要方式。除了使用 docker 命令的方式，还可以使用直接请求 REST API 的方式与 Docker 服务端交互，甚至还可以使用各种语言的 SDK 与 Docker 服务端交互。目前社区维护着 Go、Java、Python、PHP 等数十种语言的 SDK。

### Docker 服务端

Docker 服务端是 Docker 所有后台服务的统称。其中 dockerd 是一个非常重要的后台管理进程，它负责响应和处理来自 Docker 客户端的请求，然后将客户端的请求转化为 Docker 的具体操作。例如镜像、容器、网络和挂载卷等具体对象的操作和管理。

### Docker 重要组件

* runC：是 Docker 官方按照 OCI 容器运行时标准的一个实现。通俗地讲，runC 是一个用来运行容器的轻量级工具，是真正用来运行容器的。
* containerd：是 Docker 服务端的一个核心组件，他是从dockerd中剥离出来的。containerd 通过 containerd-shim 启动并管理 runC，可以说 containerd 真正管理了容器的生命周期。

![Drawing 3.png](https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYuuAQINxAAA236heaL0459.png)

通过上图可以看到，dockerd 通过 gRPC 与 containerd 通信，由于 dockerd 与真正的容器运行时 runC 中间有了 containerd 这一 OCI 标准层，使得 dockerd 可以确保接口向下兼容。

**Docker 各组件之间的关系**

dockerd 启动的时候，containerd 就随之启动了，dockerd 和 containerd 一直存在。当执行 docker run 命令时，containerd 会创建 containerd-shim 充当“垫片”进程，然后启动容器的真正进程。



### Docker 组件剖析

在 Docker 安装路径下执行 ls 命令，这样可以看到以下与 Docker 有关的组件。

```shell
containerd
containerd-shim
ctr
docker
docker-init
docker-proxy
dockerd
runc
```

从整体架构可知，Docker 组件可以根据工作职责大体分为 Docker 相关组件、containerd 相关组件和容器运行时相关组件三大类。

Docker 的组件虽然很多，但每个组件都有自己清晰的工作职责，Docker 相关的组件负责发送和接受 Docker 请求，contianerd 相关的组件负责管理容器的生命周期，而 runc 负责真正意义上创建和启动容器。这些组件相互配合，才使得 Docker 顺利完成了容器的管理工作。

#### Docker 相关的组件

包括 docker、dockerd、docker-init 和 docker-proxy。

**（1）docker**

docker 是 Docker 客户端的一个完整实现，它是一个二进制文件，对用户可见的操作形式为 docker 命令，通过 docker 命令可以完成所有的 Docker 客户端与服务端的通信。

**（2）dockerd**

dockerd 是 Docker 服务端的后台常驻进程，用来接收客户端发送的请求，执行具体的处理任务，处理完成后将结果返回给客户端。

Docker 客户端可以通过多种方式向 dockerd 发送请求，我们常用的 Docker 客户端与 dockerd 的交互方式有三种。

* 通过 UNIX 套接字与服务端通信：配置格式为unix://socket_path，默认 dockerd 生成的 socket 文件路径为 /var/run/docker.sock，该文件只有 root 用户或者 docker 用户组的用户才可以访问，这就是为什么 Docker 刚安装完成后只有 root 用户才能使用 docker 命令的原因。
* 通过 TCP 与服务端通信：配置格式为tcp://host:port，通过这种方式可以实现客户端远程连接服务端，但是在方便的同时也带有安全隐患，因此在生产环境中如果你要使用 TCP 的方式与 Docker 服务端通信，推荐使用 TLS 认证，可以通过设置 Docker 的 TLS 相关参数，来保证数据传输的安全。
* 通过文件描述符的方式与服务端通信：配置格式为：fd://这种格式一般用于 systemd 管理的系统中。

Docker 客户端和服务端的通信形式必须保持一致，否则将无法通信，只有当 dockerd 监听了 UNIX 套接字客户端才可以使用 UNIX 套接字的方式与服务端通信，UNIX 套接字也是 Docker 默认的通信方式，如果你想要通过远程的方式访问 dockerd，可以在 dockerd 启动的时候添加 -H 参数指定监听的 HOST 和 PORT。

**（3）docker-init**

在 Linux 系统中，1 号进程是 init 进程，是所有进程的父进程。主机上的进程出现问题时，init 进程可以帮我们回收这些问题进程。同样的，在容器内部，当我们自己的业务进程没有回收子进程的能力时，在执行 docker run 启动容器时可以添加 --init 参数，此时 Docker 会使用 docker-init 作为1号进程，帮你管理容器内子进程，例如回收僵尸进程等。

**（4）docker-proxy**

docker-proxy 主要是用来做端口映射的。当我们使用 docker run 命令启动容器时，如果使用了 -p 参数，docker-proxy 组件就会把容器内相应的端口映射到主机上来，底层是依赖于 iptables 实现的。

#### containerd 相关的组件

包括 containerd、containerd-shim 和 ctr。

**（1）containerd**

[containerd](https://github.com/containerd/containerd) 组件是从 Docker 1.11 版本正式从 dockerd 中剥离出来的，它的诞生完全遵循 OCI 标准，是容器标准化后的产物。containerd 完全遵循了 OCI 标准，并且是完全社区化运营的，因此被容器界广泛采用。

containerd 不仅负责容器生命周期的管理，同时还负责一些其他的功能：

- 镜像的管理，例如容器运行前从镜像仓库拉取镜像到本地；
- 接收 dockerd 的请求，通过适当的参数调用 runc 启动容器；
- 管理存储相关资源；
- 管理网络相关资源。

containerd 包含一个后台常驻进程，默认的 socket 路径为 /run/containerd/containerd.sock，dockerd 通过 UNIX 套接字向 containerd 发送请求，containerd 接收到请求后负责执行相关的动作并把执行结果返回给 dockerd。

如果你不想使用 dockerd，也可以直接使用 containerd 来管理容器，由于 containerd 更加简单和轻量，生产环境中越来越多的人开始直接使用 containerd 来管理容器。

**（2）containerd-shim**

containerd-shim 的意思是垫片，类似于拧螺丝时夹在螺丝和螺母之间的垫片。containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 containerd 不影响已经启动的容器进程。

**（3）ctr**

ctr 实际上是 containerd-ctr，它是 containerd 的客户端，主要用来开发和调试，在没有 dockerd 的环境中，ctr 可以充当 docker 客户端的部分角色，直接向 containerd 守护进程发送操作容器的请求。

了解完 containerd 相关的组件，我们来了解一下容器的真正运行时 runc。

#### 容器运行时组件 runc

runc 是一个标准的 OCI 容器运行时的实现，它是一个命令行工具，可以直接用来创建和运行容器。

#### 总结

![7.png](https://s0.lgstatic.com/i/image/M00/59/E6/Ciqc1F9y4vGAVzmAAADk1nlHpUA424.png)

### Docker 当前架构的弊端

当前 docker 的问题就是调用链过长，出现问题需要分析很多组件才能定位问题。

抽象分层就需要写时复制，性能是有一定损耗的。