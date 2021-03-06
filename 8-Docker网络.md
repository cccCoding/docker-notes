## Docker网络

### 容器网络发展史

Docker 从 1.7 版本开始，便把网络和存储从 Docker 中正式以插件的形式剥离开来，并且分别为其定义了标准。Docker 定义的网络模型标准称之为 CNM (Container Network Model) 。于此同时，CoreOS 推出了 CNI（Container Network Model）。

后来由于 Kubernetes 逐渐成为了容器编排的标准，而 Kubernetes 最终选择了 CNI 作为容器网络的定义标准（具体原因可以参考[这里](https://kubernetes.io/blog/2016/01/why-kubernetes-doesnt-use-libnetwork/)），容器的网络标准分为了两大阵营，一个是以 Docker 公司为代表的 CNM，另一个便是以 Google、Kubernetes、CoreOS 为代表的 CNI 网络标准。

#### CNM

CNM (Container Network Model) 是 Docker 发布的容器网络标准，意在规范和指定容器网络发展标准，CNM 抽象了容器的网络接口 ，使得只要满足 CNM 接口的网络方案都可以接入到 Docker 容器网络，更好地满足了用户网络模型多样化的需求。

CNM 只是定义了网络标准，对于底层的具体实现并不太关心，这样便解耦了容器和网络，使得容器的网络模型更加灵活。

CNM 定义的网络标准包含三个重要元素。

* 沙箱（Sandbox）：沙箱代表了一系列网络堆栈的配置，其中包含路由信息、网络接口等网络资源的管理，沙箱的实现通常是 Linux 的 Net Namespace，但也可以通过其他技术来实现，比如 FreeBSD jail 等。
* 接入点（Endpoint）：接入点将沙箱连接到网络中，代表容器的网络接口，接入点的实现通常是 Linux 的 veth 设备对。
* 网络（Network）：网络是一组可以互相通信的接入点，它将多个接入点组成一个子网，并且多个接入点之间可以相互通信。

CNM 的三个要素基本抽象了所有网络模型，使得网络模型的开发更加规范。

为了更好地构建容器网络标准，Docker 团队把网络功能从 Docker 中剥离出来，成为独立的项目 Libnetwork，它通过插件的形式为 Docker 提供网络功能。Libnetwork 是开源的，使用 Golang 编写，它完全遵循 CNM 网络规范，是 CNM 的官方实现。

### Libnetwork

Libnetwork 是 Docker 启动容器时，用来为 Docker 容器提供网络接入功能的插件，它可以让 Docker 容器顺利接入网络，实现主机和容器网络的互通。

#### Libnetwork 常见网络模式

Libnetwork 可以构建出多种网络模式，比较典型的网络模式主要有四种，以满足我们的在不同场景下的需求（基本满足了单机容器的所有场景）。

1. null 空网络模式：可以帮助我们构建一个没有网络接入的容器环境，以保障数据安全。

2. bridge 桥接模式：可以打通容器与容器间网络通信的需求。
3. host 主机网络模式：可以让容器内的进程共享主机网络，从而监听或修改主机网络。
4. container 网络模式：可以将两个容器放在同一个网络命名空间内，让两个业务通过 localhost 即可实现访问。

**查看 docker 网络信息**

```shell
docker network ls
```

**（1）null 空网络模式**

业务场景：处理一些保密数据，出于安全考虑，需要一个隔离的网络环境执行任务。这时候我们的容器就像一个没有联网的电脑，处于一个相对较安全的环境，确保我们的数据不被他人从网络窃取。

使用 Docker 创建 null 空网络模式的容器时，容器拥有自己独立的 Net Namespace，但是此时的容器并没有任何网络配置。在这种模式下，Docker 除了为容器创建了 Net Namespace 外，没有创建任何网卡接口、IP 地址、路由等网络配置。

我们使用`docker run`命令启动时，添加 --net=none 参数即可启动一个空网络模式的容器：

```shell
docker run --net=none -it busybox
```

**（2）bridge 桥接模式**

**bridge 桥接模式是 Docker 启动容器时默认的网络模式**，使用 bridge 网络可以实现容器与容器的互通，一个容器可以通过容器 IP 访问到另外一个容器。同时使用 bridge 网络可以实现主机与容器的互通，我们在容器内启动的业务，可以从主机直接请求。

Docker 的 bridge 模式是由 Linux 的 veth 和 bridge 这两种技术实现的。

* Linux veth

veth 是 Linux 中的虚拟设备接口，veth 都是成对出现的，它在容器中充当一个桥梁。veth 可以用来连接虚拟网络设备，例如 veth 可以用来连通两个 Net Namespace，从而使得两个 Net Namespace 之间可以互相访问。

* Linux bridge

Linux bridge 是一个虚拟设备，是用来连接网络的设备，相当于物理网络环境中的交换机。Linux bridge 可以用来转发两个 Net Namespace 内的流量。

* veth 与 bridge 的关系

![Lark20200929-162853.png](https://s0.lgstatic.com/i/image/M00/59/ED/Ciqc1F9y8IKAa-1NAABjDM-2kBk665.png)

通过图 1 可以看到，bridge 就像一台交换机，而 veth 就像一根网线，通过交换机和网线可以把两个不同 Net Namespace 的容器连通，使得它们可以互相通信。

**Docker 的 bridge 模式也是这种原理。Docker 启动时，Libnetwork 会在主机上创建 docker0 网桥，docker0 网桥就相当于图 1 中的交换机，而 Docker 创建出的 bridge 模式的容器则都会连接 docker0 上，从而实现网络互通。**

**（3）host 主机网络模式**

业务场景：容器需要控制主机网络或者用主机网络提供服务。适用于想要使用主机网络，但又不想把运行环境直接安装到主机上的场景中。

使用 host 主机网络模式时：

* Libnetwork 不会为容器创建新的 Net Namespace 和网络配置。

* Docker 容器中的进程直接共享主机的网络配置，可以直接使用主机的网络信息。此时，在容器内监听的端口，也将直接占用到主机的端口。
* 除了网络共享主机的网络外，其他的包括进程、文件系统、主机名等都是与主机隔离的。

启动一个主机网络模式的 busybox 镜像：

```shell
docker run -it --net=host busybox
```

**（4）container 网络模式**

container 网络模式允许一个容器共享另一个容器的网络命名空间。当两个容器需要共享网络，但其他资源仍然需要隔离时就可以使用 container 网络模式。

启动一个 busybox1 容器：

```shell
docker run -d --name=busybox1 busybox sleep 3600
```

新打开一个命令行窗口，再启动一个 busybox2 容器，通过 container 网络模式连接到 busybox1 的网络：

```
docker run -it --net=container:busybox1 --name=busybox2 busybox sh
```

进入两个容器，通过`ifconfig`命令可以看到，busybox2 与 busybox1 的网络一致，IP 一致。两个容器之间可以直接通过 localhost 通信。

**小结**

![Lark20200929-162901.png](https://s0.lgstatic.com/i/image/M00/59/ED/Ciqc1F9y8HGAaH1iAAClKDUq5FY736.png)

### docker0

Docker 的 bridge 桥接模式。Docker 启动时，Libnetwork 会在主机上创建 docker0 网桥，docker0 网桥就相当于交换机，而 Docker 创建出的 bridge 模式的容器则都会连接 docker0 上，从而实现网络互通。

查看网卡信息 docker0 地址

```shell
ip addr
```

**docker 是如何处理容器网络访问的**

```shell
docker run -d -P --name tomcat01 tomcat

docker exec -it tomcat01 ip addr	# 可以看到类似 eth0@if262 的ip地址，是docker分配的
```

**宿主机和容器之间、各个容器之间能通过 ip ping 通**

原理：

1. Docker 启动时，Libnetwork 会在主机上创建 docker0 网桥，而我们每启动一个 docker 容器，docker 就会给容器分配一个 ip
2. 使用 veth 技术，通过 docker0 连通主机

**各个容器之间能通过 ip ping 通**

原理：Docker 启动时，Libnetwork 会在主机上创建 docker0 网桥，docker0 网桥就相当于交换机，而 Docker 创建出的 bridge 模式的容器则都会连接 docker0 上，从而实现网络互通。

tip：

1. docker 中所有的网络接口都是虚拟的。虚拟的转发效率高。
2. 删除容器，对应的 veth-pair ip 也删除

**各个容器之间不能通过容器名互相访问**

docker0 不支持容器之间通过容器名互相访问。

解决方案

*  --link

  原理为在hosts文件中增加映射，不推荐使用

* 自定义网络

### 自定义网络

#### 创建并使用自定义网络

创建自定义网络

```shell
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
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
docker exec -it tomcat-01 ping tomcat02
```

推荐使用自定义的网络，docker 帮我们维护好了对应的关系。

比如，为 redis 集群和 mysql 集群创建网络，相同自定义网络下可以连通，不同自定义网络间相互隔离。

#### 网络连通

一个在 A 网络中的容器连接到另一个网络 B

```shell
# docker0 网络下创建 tomcat03
docker run -d -P --name tomcat03 tomcat

# tomcat03 连接自定义网络 mynet
docker network connect mynet tomcat03

# 查看自定义网络，发现 tomcat03 加入到自定义网络 mynet 中。一个容器，两个ip。
docker network inspect mynet

# 测试
docker exec -it tomcat03 ping tomcat02		# 可以ping通
```

