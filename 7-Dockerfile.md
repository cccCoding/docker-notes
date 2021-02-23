## Dockerfile

### 概述

Dockerfile 是一个包含了用户所有构建命令的文本。通过 `docker build` 命令可以根据 Dockerfile 构建镜像。

### 特性

使用 Dockerfile 构建镜像具有以下特性：

1. Dockerfile 的每一行命令都会生成一个独立的镜像层，并且拥有唯一的 ID
2. Dockerfile 的命令是完全透明的，Dockerfile 的内容指明了如何一步步构建镜像
3. Dockerfile 是纯文本的，方便跟随代码一起存放在代码仓库并做版本管理

### 优点

生产实践中，优先使用 Dockerfile 的方式构建镜像。使用 Dockerfile 构建镜像可以带来很多好处：

* 易于版本化管理：Dockerfile 本身是个文本文件，方便存放在代码仓库做版本管理，可以很方便地找到各个版本之间的变更历史
* 过程可追溯：Dockerfile 的每一行指令代表一个镜像层，根据 Dockerfile 的内容即可明确地查看完整构建过程
* 屏蔽构建环境异构：使用 Dockerfile 构建镜像无需考虑构建环境，基于相同 Dockerfile 无论在哪运行，构建结果都一致

### 语法

1. 每个保留关键字（指令）都必须是大写字母
2. 指令从上到小顺序执行
3. #表示注释

### 指令

* FORM：基础镜像。Dockerfile 除了注释第一行必须是 FROM，FROM 后面跟镜像名称，代表我们要基于哪个基础镜像构建我们的容器。大部分镜像都是从 scratch 这个最基础的镜像过来的。
* MAINTAINER：维护者信息，姓名+邮箱
* ADD：COPY 本机文件或远程文件到镜像内，会自动解压
* COPY：拷贝本地文件到镜像内
* RUN：后面跟一个具体的命令，镜像构建时执行
* ENV：指定容器运行时的环境变量，格式为 key=value
* WORKDIR：为 Dockerfile 中跟在其后的所有 RUN、CMD、ENTRYPOINT、COPY 和 ADD 命令设置工作目录。
* VOLUME：设置卷，挂载主机目录
* EXPOSE：指定容器监听的端口
* CMD：指定容器启动时执行的命令，只有最后一个会生效，可被替代
* ENTRYPOINT：指定容器启动时执行的命令，可以追加命令
* ONBUILD： 当构建一个被继承的 Dockerfile，就会触发运行

### 如何在生产中编写最优 Dockerfile

#### Dockerfile 书写原则

1. 单一职责：容器的本质是进程，一个容器代表一个进程，因此不同功能的应用应该尽量拆分为不同的容器，每个容器只负责单一业务进程。
2. 提供注释信息
3. 保持容器最小化：避免安装无用的软件包，加快容器构建速度，避免镜像体积过大。
4. 合理选用基础镜像
5. 使用 .dockerignore 文件
6. 尽量使用构建缓存
7. 正确设置时区
8. 使用国内软件源加快镜像构建速度
9. 最小化镜像层数

#### Docker 指令书写建议

* RUN

  RUN 指令在构建时将会生成一个新的镜像层并且执行 RUN 指令后面的内容。

  使用 RUN 指令时应该尽量遵循以下原则：

  * 当 RUN指令后面跟的内容比较复杂时，建议使用反斜杠（\） 结尾并且换行；
  * RUN 指令后面的内容尽量按照字母顺序排序，提高可读性。

* CMD 和 ENTRYPOINT

  CMD 和 ENTRYPOINT 指令都是容器运行的命令入口。

  **相同之处**，CMD 和 ENTRYPOINT的 基本使用格式分为两种。

  * 第一种为 CMD/ENTRYPOINT["command" , "param"]。这种格式是使用 Linux 的 exec 实现的， 一般称为 exec 模式，这种书写格式为 CMD/ENTRYPOINT 后面跟 json 数组，也是 Docker 推荐的使用格式。
  * 另一种格式为 CMD/ENTRYPOINT command param，这种格式是基于 shell 实现的，通常称为 shell 模式。当使用 shell 模式时，Docker 会以 `bin/sh -c command`的方式执行命令。

  > 使用 exec 模式启动容器时，容器的 1 号进程就是 CMD/ENTRYPOINT 中指定的命令，而使用 shell 模式启动容器时相当于我们把启动命令放在了 shell 进程中执行，等效于执行 /bin/sh -c "task command" 命令。因此 shell 模式启动的进程在容器中实际上并不是 1 号进程。

  **不同之处**

  * Dockerfile 中如果使用了 ENTRYPOINT 指令，启动 Docker 容器时需要使用 --entrypoint 参数才能覆盖 Dockerfile 中的 ENTRYPOINT 指令，而使用 CMD 设置的命令则可以被 docker run 后面的参数直接覆盖。
  * ENTRYPOINT 指令可以结合 CMD 指令使用，也可以单独使用，而 CMD 指令只能单独使用。

  **怎样选择**

  如果希望你的镜像足够灵活，推荐使用 CMD 指令。如果你的镜像只执行单一的具体程序，并且不希望用户在执行 docker run 时覆盖默认程序，建议使用 ENTRYPOINT。

  **注意**，无论使用 CMD 还是 ENTRYPOINT，都尽量使用 exec 模式。

* ADD 和 COPY

  ADD 和 COPY 指令功能类似，都是从外部往容器内添加文件。但是 COPY 指令只支持基本的文件和文件夹拷贝功能，ADD 则支持更多文件来源类型，比如自动提取 tar 包，并且可以支持源文件为 URL 格式。

  在日常应用中，推荐使用 COPY 指令，因为 COPY 指令更加透明，仅支持本地文件向容器拷贝，而且使用COPY 指令可以更好地利用构建缓存，有效减小镜像体积。

* WORKDIR

  为了使构建过程更加清晰明了，推荐使用 WORKDIR 来指定容器的工作路径，应该尽量避免使用`RUN cd /work/path && do some work`这样的指令。

#### Dockerfile 镜像多阶级构建

Docker 镜像是分层的，每一层镜像都会额外占用存储空间，一个 Docker 镜像层数越多，这个镜像占用的存储空间则会越多。镜像构建最重要的一个原则就是要保持镜像体积尽可能小，要实现这个目标通常可以从两个方面入手：

* 基础镜像体积应该尽量小；
* 尽量减少 Dockerfile 的行数，因为 Dockerfile 的每一条指令都会生成一个镜像层。

为减少 Dockerfile 的行数，可以将镜像的编译过程和运行过程分开。可以使用镜像多阶级构建（multistage-build）实现。

**限制条件**

使用多阶级构建的唯一限制条件是 Docker 版本必须高于 17.05 。

**使用多阶级构建**

Docker 允许我们在 Dockerfile 中使用多个 FROM 语句，每个 FROM 语句都可以使用不同基础镜像。最终生成的镜像，以最后一条 FROM 为准。所以我们可以在一个 Dockerfile 中声明多个 FROM，然后选择性地将一个阶段生成的文件拷贝到另外一个阶段中，从而实现最终的镜像只保留我们需要的环境和文件。多阶段构建的主要使用场景是分离编译环境和运行环境。

```
FROM golang:1.13
WORKDIR /go/src/github.com/wilhelmguo/multi-stage-demo/
COPY main.go .
RUN CGO_ENABLED=0 GOOS=linux go build -o http-server .
FROM alpine:latest  
WORKDIR /root/
COPY --from=0 /go/src/github.com/wilhelmguo/multi-stage-demo/http-server .
CMD ["./http-server"] 
```

以上是 Dockerfile 的内容，使用第二个 FROM 命令表示镜像构建的第二阶段，使用 COPY 指令拷贝编译后的文件到 alpine 镜像中，--from=0 表示从第一阶段构建结果中拷贝文件到当前构建阶段。

然后通过命令可实现镜像的构建：

```
docker build -t http-server:latest .
```

**为构建阶段命名**

默认情况下，每一个构建阶段都没有被命名，你可以通过 FROM 指令出现的顺序来引用这些构建阶段，构建阶段的序号是从 0 开始的。然而，为了提高 Dockerfile 的可读性，我们需要为某些构建阶段起一个名称，这样即便后面我们对 Dockerfile 中的内容进程重新排序或者添加了新的构建阶段，其他构建过程中的 COPY 指令也不需要修改。

优化 Dockerfile

```
FROM golang:1.13 AS builder
WORKDIR /go/src/github.com/wilhelmguo/multi-stage-demo/
COPY main.go .
RUN CGO_ENABLED=0 GOOS=linux go build -o http-server .
FROM alpine:latest  
WORKDIR /root/
COPY --from=builder /go/src/github.com/wilhelmguo/multi-stage-demo/http-server .
CMD ["./http-server"]
```

在第一个构建阶段，使用 AS 指令将这个阶段命名为 builder。然后在第二个构建阶段使用 --from=builder 指令，即可从第一个构建阶段中拷贝文件，使得 Dockerfile 更加清晰可读。

**停止在特定的构建阶段**

有时候，我们的构建阶段非常复杂，我们想在代码编译阶段进行调试，但是多阶段构建默认构建 Dockerfile 的所有阶段，为了减少每次调试的构建时间，我们可以使用 target 参数来指定构建停止的阶段。

例如，我只想在编译阶段调试 Dockerfile 文件，可以使用如下命令：

```shell
$ docker build --target builder -t http-server:latest .
```

**使用现有镜像作为构建阶段**

使用多阶段构建时，不仅可以从 Dockerfile 中已经定义的阶段中拷贝文件，还可以使用 COPY --from 指令从一个指定的镜像中拷贝文件，指定的镜像可以是本地已经存在的镜像，也可以是远程镜像仓库上的镜像。

例如，当我们想要拷贝 nginx 官方镜像的配置文件到我们自己的镜像中时，可以在 Dockerfile 中使用以下指令：

```
COPY --from=nginx:latest /etc/nginx/nginx.conf /etc/local/nginx.conf
```

从现有镜像中拷贝文件还有一些其他的使用场景。例如，有些工具没有我们使用的操作系统的安装源，或者安装源太老，需要我们自己下载源码并编译这些工具，但是这些工具可能依赖的编译环境非常复杂，而网上又有别人已经编译好的镜像。这时我们就可以使用 COPY --from 指令从编译好的镜像中将工具拷贝到我们自己的镜像中。

### Dockerfile 构建镜像发布步骤

1. 编写一个 Dockerfile 文件
2. docker build 构建镜像
3. docker run 运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库）

### 实战：构建一个自己的 centos

**1. 编写 Dockerfile 文件**

```shell
FROM centos
MAINTAINER cccCoding<123@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash
```

**2.根据Dockerfile构建镜像**

```shell
docker build -t mycentos:1.0 .
```

**3.测试运行**

```shell
docker run -it mycentos:1.0

# 进入容器执行
pwd		# 在工作目录 /usr/local

ifconfig	# 可以使用
vim test	# 可以使用
```

查看本地镜像构建历史

```shell
docker history 镜像id
```

### 小结

Docker 镜像逐渐成为企业应用交付的标准，必须要掌握。

Dockerfile 是面向开发的，我们以后要发布项目，做镜像，就需要编写 Dockerfile 文件。