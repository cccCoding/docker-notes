## Dockerfile

#### 概述

Dockerfile就是用来构建docker镜像的文件。一个命令参数脚本。

构建步骤：

1. 编写一个dockerfile文件
2. docker build 构建成为镜像
3. docker run 运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库）

#### 基础知识

1. 每个保留关键字（指令）都必须是大写字母
2. 指令从上到小顺序执行
3.  #表示注释
4. 每一个指令都会创建一个新的镜像层，并提交

dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件。

Docker镜像逐渐成为企业交付的标准，必须要掌握。

#### Dockerfile 指令

```shell
# FORM

```

