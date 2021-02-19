## Docker概述

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

### Docker架构

Docker 采用 C/S （客户端/服务端）架构。客户端负责发送操作指令，服务端负责接收和处理指令。客户端和服务端通信有多种方式，既可以在同一台机器上通过UNIX套接字通信，也可以通过网络连接远程通信。

#### **架构图**

![Drawing 2.png](https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYtCAC1GSAADIK4E6wrc368.png)

#### Docker 客户端

Docker 客户端其实是一种泛称。其中 docker 命令是 Docker 用户与 Docker 服务端交互的主要方式。除了使用 docker 命令的方式，还可以使用直接请求 REST API 的方式与 Docker 服务端交互，甚至还可以使用各种语言的 SDK 与 Docker 服务端交互。目前社区维护着 Go、Java、Python、PHP 等数十种语言的 SDK。

#### Docker 服务端

Docker 服务端是 Docker 所有后台服务的统称。其中 dockerd 是一个非常重要的后台管理进程，它负责响应和处理来自 Docker 客户端的请求，然后将客户端的请求转化为 Docker 的具体操作。例如镜像、容器、网络和挂载卷等具体对象的操作和管理。

#### Docker 重要组件

* runC：是 Docker 官方按照 OCI 容器运行时标准的一个实现。通俗地讲，runC 是一个用来运行容器的轻量级工具，是真正用来运行容器的。
* containerd：是 Docker 服务端的一个核心组件，他是从dockerd中剥离出来的。containerd 通过 containerd-shim 启动并管理 runC，可以说 containerd 真正管理了容器的生命周期。

![Drawing 3.png](https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYuuAQINxAAA236heaL0459.png)

通过上图可以看到，dockerd 通过 gRPC 与 containerd 通信，由于 dockerd 与真正的容器运行时 runC 中间有了 containerd 这一 OCI 标准层，使得 dockerd 可以确保接口向下兼容。

**Docker 各组件之间的关系**

dockerd 启动的时候，containerd 就随之启动了，dockerd 和 containerd 一直存在。当执行 docker run 命令时，containerd 会创建 containerd-shim 充当“垫片”进程，然后启动容器的真正进程。