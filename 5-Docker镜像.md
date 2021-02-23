## Docker 镜像

### **简介**

镜像是一个只读的容器模板，包含启动容器所需要的所有文件系统结构和内容。简单来讲，镜像是一个特殊的文件系统，它提供了容器运行时所需的程序、软件库、资源、配置等静态数据。镜像不包含任何动态数据，镜像内容在构建后不会被改变。

### **如何得到镜像**

* 拉取远程仓库镜像
* 镜像文件导出镜像
* 使用 docker build 命令基于 DockerFile 构建镜像
* 使用 docker commit 命令基于已经运行的容器提交镜像

### 镜像实现原理

Docker 镜像由一系列镜像层（layer）组成，每一层代表了镜像构建过程中的一次提交。

进入 /var/lib/docker/overlay2 目录，使用`tree .`命令查看镜像文件，通过目录结构可以看到，Dockerfile 的每一行命令，都生成了一个镜像层，每一层的 diff 文件夹下只存放了增量数据。

![Lark20200904-175137.png](https://s0.lgstatic.com/i/image/M00/4A/AD/CgqCHl9SDmGACBEjAABkgtnn_hE625.png)

分层的结构使得 Docker 镜像非常轻量，每一层根据镜像的内容都有一个唯一的 ID 值，当不同的镜像之间有相同的镜像层时，便可以实现不同的镜像之间共享镜像层的效果。下载镜像时可以看到分层下载的过程。

总结一下， Docker 镜像是静态的分层管理的文件组合，镜像底层的实现依赖于联合文件系统。

### **镜像加载原理**

#### UnionFS（联合文件系统）

是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层地叠加，同时可以将不同目录挂载到同一个虚拟文件系统下，使用户只能看到一个文件系统。联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像，可以制作各种具体的应用镜像。

#### Docker 镜像加载原理

**bootfs（boot file system）：**主要包含 bootloader 和 kernel，bootloader 主要是引导加载 kernel，Linux 刚启动时会加载 bootfs 文件系统，在 docker 镜像的最底层是 bootfs。这一层与我们经典的 Linux/Unix 系统是一样的，包含 boot 加载器和内核。当 boot 加载完成之后整个内核就都在内存中了，此时内存的使用权已由 bootfs 转交给内核，系统也会卸载 bootfs。

**rootfs（root file system）：**在 bootfs 之上，包含的就是典型 Linux 系统中的 /dev，/proc，/bin，/etc 等标准目录和文件。rootfs 就是各种不同的操作系统发行版，如 Ubuntu、CentOS 等。

CentOS 镜像只有200多M，是因为对于一个精简的 OS，rootfs 可以很小，只需要包含最基本的命令、工具和程序库就可以了，因为底层直接用 Host 的 kernel，自己只需要提供 rootfs 就可以了。由此可见对于不同的 Linux 发行版，bootfs 基本是一致的，rootfs 会有差别，因此不同的发行版可以共用 bootfs。

### commit 镜像

```shell
# 提交容器成为一个新的镜像，保存当前容器的状态
docker commit -m "提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

### 发布镜像到 DockerHub

1. https://hub.docker.com/  注册账号

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