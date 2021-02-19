## Docker镜像

### **简介**

镜像是一个只读的容器模板，包含启动容器所需要的所有文件系统结构和内容。简单来讲，镜像是一个特殊的文件系统，它提供了容器运行时所需的程序、软件库、资源、配置等静态数据。镜像不包含任何动态数据，镜像内容在构建后不会被改变。

### **如何得到镜像**

* 拉取远程仓库镜像
* 镜像文件导出镜像
* 使用 docker build 命令基于 DockerFile 构建镜像
* 使用 docker commit 命令基于已经运行的容器提交为镜像

### 镜像实现原理

Docker 镜像由一系列镜像层（layer）组成，每一层代表了镜像构建过程中的一次提交。

进入 /var/lib/docker/overlay2 目录，使用 tree . 命令查看镜像文件，通过目录结构可以看到，Dockerfile 的每一行命令，都生成了一个镜像层，每一层的 diff 文件夹下只存放了增量数据。

![Lark20200904-175137.png](C:\Users\Administrator\Desktop\容器\docker-notes\5-Docker镜像.assets\CgqCHl9SDmGACBEjAABkgtnn_hE625.png)

分层的结构使得 Docker 镜像非常轻量，每一层根据镜像的内容都有一个唯一的 ID 值，当不同的镜像之间有相同的镜像层时，便可以实现不同的镜像之间共享镜像层的效果。

总结一下， Docker 镜像是静态的分层管理的文件组合，镜像底层的实现依赖于联合文件系统（UnionFS）。

### **镜像加载原理**

> UnionFS（联合文件系统）

是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层地叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像，可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

下载时可以看到一层层地进行下载。

> Docker镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统就是UnionFS。

bootfs（boot file system）：主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在docker镜像的最底层是bootfs。这一层与我们经典的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，系统也会卸载bootfs。

rootfs（root file system）：在bootfs之上，包含的就是典型Linux系统中的/dev，/proc，/bin，/etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，如Ubuntu、CentOS等。

CentOS镜像只有200多M，是因为对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供rootfs就可以了。由此可见对于不同的Linux发行版，bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以共用bootfs。

#### **分层理解**

docker镜像都是只读的，当容器启动时，一个新的可写成被加载到镜像的顶部。这一层就是我们通常说的容器层，容器层之下都叫镜像层。

#### commit镜像

```shell
# 提交容器成为一个新的镜像，保存当前容器的状态
docker commit -m "提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

