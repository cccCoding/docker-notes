## Dockerfile

### 概述

Dockerfile 是一个包含了用户所有构建命令的文本。通过 docker build 命令可以从 Dockerfile 构建镜像。

### 特性

使用 Dockerfile 构建镜像具有以下特性：

1. Dockerfile 的每一行命令都会生成一个独立的镜像层，并且拥有唯一的 ID
2. Dockerfile 的命令是完全透明的，Dockerfile 的内容指明了如何一步步构建镜像
3. Dockerfile 是纯文本的，方便跟随代码一起存放在代码仓库并做版本管理

### 语法

1. 每个保留关键字（指令）都必须是大写字母
2. 指令从上到小顺序执行
3. #表示注释

### 指令

* FORM：基础镜像。Dockerfile 除了注释第一行必须是 FROM ，FROM 后面跟镜像名称，代表我们要基于哪个基础镜像构建我们的容器。大部分镜像都是从scratch这个最基础的镜像过来的
* MAINTAINER：维护者信息，姓名+邮箱
* ADD：COPY 本机文件或远程文件到镜像内，会自动解压
* COPY：拷贝本地文件到镜像内
* RUN：后面跟一个具体的命令，镜像构建的时执行
* ENV：指定容器运行时的环境变量，格式为 key=value
* WORKDIR：为 Dockerfile 中跟在其后的所有 RUN、CMD、ENTRYPOINT、COPY 和 ADD 命令设置工作目录。
* VOLUME：设置卷，挂载主机目录
* EXPOSE：指定容器监听的端口
* CMD：指定容器启动时执行的命令，只有最后一个会生效，可被替代
* ENTRYPOINT：指定容器启动时执行的命令，可以追加命令
* ONBUILD 当构建一个被继承的Dockerfile，就会触发运行

### 如何在生产中编写最优 Dockerfile

生产实践中，优先使用 Dockerfile 的方式构建镜像。使用 Dockerfile 构建镜像可以带来很多好处：

* 易于版本化管理：Dockerfile 本身是个文本文件，方便存放在代码仓库做版本管理，可以很方便地找到各个版本之间的变更历史
* 过程可追溯：Dockerfile 的每一行指令代表一个镜像层，根据 Dockerfile 的内容即可明确地查看完整构建过程
* 屏蔽构建环境异构：使用 Dockerfile 构建镜像无需考虑构建环境，基于相同 Dockerfile 无论在哪运行，构建结果都一致

#### Dockerfile 书写原则

1. 单一职责：
2. 提供注释信息：
3. 保持容器最小化
4. 合理选用基础镜像
5. 使用 .dockerignore 文件
6. 尽量使用构建缓存
7. 正确设置时区
8. 使用国内软件源加快镜像构建速度
9. 最小化镜像层数

#### Docker 指令书写建议

1. RUN
2. CMD 和 ENTRYPOINT
3. ADD 和 COPY
4. WORKDIR

### Dockerfile 构建镜像发布步骤

1. 编写一个 Dockerfile 文件
2. docker build 构建成为镜像
3. docker run 运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库）

### 实战：构建一个自己的centos

**1. 编写Dockerfile文件 Dockerfile-mycentos**

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
docker build -f Dockerfile-mycentos -t mycentos:1.0 .
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

### 发布镜像到DockerHub

1.  https://hub.docker.com/  注册账号

2. 登陆账号

   ```shell
   docker login -u username
   ```

3. 提交镜像

   ```shell
   docker tag dfasd234asdf 作者名/镜像名:tag
   docker push 作者名/镜像名:tag
   ```

### 发布镜像到阿里云镜像服务



### 小结

Docker 镜像逐渐成为企业交付的标准，必须要掌握。

Dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写 Dockerfile 文件。