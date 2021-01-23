## Docker Compose

#### 介绍

用来定义、运行多个容器的工具。使用YAML配置文件。通过简单的命令，根据配置创建和启动所有的服务。

一个项目通常包含多个服务，通过Docker Compose 一键启动。相较于Docker Swarm，还是单机部署项目。

三个步骤：

* Dockerfile，保证项目能在任何地方运行
* docker-compose.yml，
* 运行 docker-compose up，启动项目

#### 安装

curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

#### docker-compose.yml

https://docs.docker.com/compose/compose-file/

