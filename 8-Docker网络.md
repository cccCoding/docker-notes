## Docker网络

#### docker0

```shell
# 清空环境
docker rmi -f $(docker images -aq)

# 查看网卡信息 docker0地址
ip addr

docker network ls		# bridge
```

**docker 是如何处理容器网络访问的**

```shell
docker run -d -P --name tomcat01 tomcat

docker exec -it tomcat01 ip addr	# 可以看到类似 eth0@if262 的ip地址，是docker分配的
```

**宿主机能ping通容器，容器能ping通宿主机**

> 原理

1. 我们每启动一个docker容器，docker就会给容器分配一个ip，而我们只要安装了docker，就会有一个网卡docker0
2. 桥接模式，使用的是 veth-pair 技术，veth-pair 就是一对虚拟设备接口，成对出现

**各个容器互相也能ping通**

> 原理

不指定 --net ，默认走docker0网络，通过veth-pair连接到docker0，再由docker0路由（docker会给我们的容器分配一个默认的可用ip）

tip：

1. docker 中所有的网络接口都是虚拟的。虚拟的转发效率高。
2. 删除容器，对应的veth-pair ip也删除

**容器之间通过容器名互相访问**

docker0不支持容器之间通过容器名互相访问

解决方案

*  --link

  在hosts文件中增加映射，不推荐使用

* 自定义网络

#### 自定义网络

**网络模式**

* bridge（默认）：桥接，就死docker0
* none ：不配置网络
* host：和宿主机共享网络
* container：容器内网络连通（用的少，局限大）

**创建网络**

```shell
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

docker network inspect mynet
```

在自定义网络里启动容器

```shell
docker run -d -P --name tomcat-01 --net mynet tomcat
docker run -d -P --name tomcat-02 --net mynet tomcat
```

再查看自定义网络

```shell
docker network inspect mynet
```

**在自定义网络中，容器间可以通过容器名互相访问**

```shell
docker exec -it tomcat-01 ping tomcat02		# 可以ping通
```

自定义的网络，docker都已经帮我们维护好了对应的关系。推荐使用自定义网络。

比如，为redis集群和mysql集群创建网络，相同自定义网络下可以连通，不同自定义网络间相互隔离。

#### 网络连通

一个在A网络中的容器连接到另一个网络

```shell
# docker0网络下创建tomcat03
docker run -d -P --name tomcat03 tomcat

# tomcat03连接自定义网络mynet
docker network connect mynet tomcat03

# 查看自定义网络，发现tomcat03加入到自定义网络mynet中。一个容器，两个ip。
docker network inspect mynet

# 测试
docker exec -it tomcat03 ping tomcat02		# 可以ping通！
```

