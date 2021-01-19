## Dockerfile

#### 概述

Dockerfile就是用来构建docker镜像的文件。一个命令参数脚本。

构建步骤：

1. 编写一个Dockerfile文件
2. docker build 构建成为镜像
3. docker run 运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库）

#### 基础知识

1. 每个保留关键字（指令）都必须是大写字母
2. 指令从上到小顺序执行
3.  #表示注释
4. 每一个指令都会创建一个新的镜像层，并提交

Dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写Dockerfile文件。

Docker镜像逐渐成为企业交付的标准，必须要掌握。

#### Dockerfile 指令

```shell
# FORM 基础镜像，一切从这里开始构建	# 大部分镜像都是从scratch这个最基础的镜像过来的

# MAINTAINER 维护者信息，姓名+邮箱

# COPY 类似ADD，把文件拷贝到镜像中

# ADD COPY文件，会自动解压

# RUN 镜像构建的时候需要运行的命令

# ENV 构建的时候设置环境变量

# WORKDIR 当前工作目录

# VOLUME 设置卷，挂载主机目录

# EXPOSE 指定对外端口

# CMD 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代

# ENTRYPOINT 指定这个容器启动的时候要运行的命令，可以追加命令

# ONBUILD 当构建一个被继承的Dockerfile，就会触发运行
```

#### 构建一个自己的centos

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

#### 发布镜像到DockerHub

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

#### 发布镜像到阿里云镜像服务

