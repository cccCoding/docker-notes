## Docker 技术原理

Docker 是基于 Linux 的 Namespace 、Cgroups 和联合文件系统这三大核心技术实现的， 即使用 Namespace 做主机名、网络、PID 等资源的隔离，使用 Cgroups 对进程或者进程组做资源（例如：CPU、内存等）的限制，联合文件系统用于镜像构建和容器运行环境。

### chroot

**什么是 chroot**

> chroot 是在 Unix 和 Linux 系统的一个操作，针对正在运作的软件进程和它的子进程，改变它外显的根目录。一个运行在这个环境下，经由 chroot 设置根目录的程序，它不能够对这个指定根目录之外的文件进行访问动作，不能读取，也不能更改它的内容。

通俗地讲，chroot 可以把任何目录更改为当前进程的根目录，使这个程序不能访问目录之外的其他目录。

chroot 是最早的容器雏形。

### Namespace

Docker 是使用 Linux 的 Namespace 技术实现各种资源隔离的。

那么什么是 Namespace，各种 Namespace 都有什么作用，为什么 Docker 需要 Namespace呢？

#### 定义

> Namespace 是 Linux 内核的一项功能，该功能对内核资源进行分区，以使一组进程看到一组资源，而另一组进程看到另一组资源。Namespace 的工作方式通过为一组资源和进程设置相同的 Namespace 而起作用，但是这些 Namespace 引用了不同的资源。资源可能存在于多个 Namespace 中。这些资源可以是进程 ID、主机名、用户 ID、文件名、与网络访问相关的名称和进程间通信。

简单来说，Namespace 是 Linux 内核的一个特性，该特性可以实现在同一主机系统中，对进程 ID、主机名、用户 ID、文件名、网络和进程间通信等资源的隔离。Docker 利用 Linux 内核的 Namespace 特性，实现了每个容器的资源相互隔离，从而保证容器内部只能访问到自己 Namespace 的资源。

最新的 Linux 5.6 内核中提供了 8 种类型的 Namespace：

| Namespace 名称                   | 作用                           | 内核版本 |
| -------------------------------- | ------------------------------ | -------- |
| Mount（mnt）                     | 隔离文件系统挂载点             | 2.4.19   |
| Process ID (pid)                 | 隔离进程 ID                    | 2.6.24   |
| Network (net)                    | 隔离网络设备，端口号等         | 2.6.29   |
| Interprocess Communication (ipc) | 隔离信号量、消息队列和共享内存 | 2.6.19   |
| UTS Namespace(uts)               | 隔离主机名和域名               | 2.6.19   |
| User Namespace (user)            | 隔离用户和用户组               | 3.8      |
| Control group (cgroup) Namespace | 隔离 Cgroups 根目录            | 4.6      |
| Time Namespace                   | 隔离系统时间                   | 5.6      |

虽然 Linux 内核提供了8种 Namespace，但是最新版本的 Docker 只使用了其中的前6 种，分别为Mount Namespace、PID Namespace、Net Namespace、IPC Namespace、UTS Namespace、User Namespace。

#### 各种 Namespace 的作用

**（1）Mount Namespace**

用来隔离不同的进程或进程组看到的挂载点。通俗地说，就是可以实现在不同的进程中看到不同的挂载目录。使用 Mount Namespace 可以实现容器内只能看到自己的挂载信息，在容器内的挂载操作不会影响主机的挂载目录。

**（2）PID Namespace**

PID Namespace 的作用是用来隔离进程。在不同的 PID Namespace 中，进程可以拥有相同的 PID 号，利用 PID Namespace 可以实现每个容器的主进程为 1 号进程，而容器内的进程在主机上却拥有不同的PID。例如一个进程在主机上 PID 为 122，使用 PID Namespace 可以实现该进程在容器内看到的 PID 为 1。

**（3）UTS Namespace**

UTS Namespace 主要是用来隔离主机名的，它允许每个 UTS Namespace 拥有一个独立的主机名。例如我们的主机名称为 docker，使用 UTS Namespace 可以实现在容器内的主机名称为 lagoudocker 或者其他任意自定义主机名。

**（4）IPC Namespace**

IPC Namespace 主要是用来隔离进程间通信的。例如 PID Namespace 和 IPC Namespace 一起使用可以实现同一 IPC Namespace 内的进程彼此可以通信，不同 IPC Namespace 的进程却不能通信。

**（5）User Namespace**

User Namespace 主要是用来隔离用户和用户组的。一个比较典型的应用场景就是在主机上以非 root 用户运行的进程可以在一个单独的 User Namespace 中映射成 root 用户。使用 User Namespace 可以实现进程在容器内拥有 root 权限，而在主机上却只是普通用户。

User Namesapce 的创建是可以不使用 root 权限的。

**（6）Net Namespace**

Net Namespace 是用来隔离网络设备、IP 地址和端口等信息的。Net Namespace 可以让每个进程拥有自己独立的 IP 地址，端口和网卡信息。例如主机 IP 地址为 172.16.4.1 ，容器内可以设置独立的 IP 地址为 192.168.1.1。

#### 为什么 Docker 需要 Namespace

当 Docker 新建一个容器时， 它会创建这六种 Namespace，然后将容器中的进程加入这些 Namespace 之中，使得 Docker 容器中的进程只能看到当前 Namespace 中的系统资源。

正是由于 Docker 使用了 Linux 的这些 Namespace 技术，才实现了 Docker 容器的隔离，可以说没有 Namespace，就没有 Docker 容器。

### Cgroups

我们知道使用不同的 Namespace，可以实现容器中的进程看不到别的容器的资源，但是还有一个问题是，容器内的进程仍然可以任意地使用主机的 CPU 、内存等资源，如果某一个容器使用的主机资源过多，可能导致主机的资源竞争，进而影响业务。那如果我们想限制一个容器资源的使用（如 CPU、内存等）应该如何做呢？

这里就需要用到 Linux 内核的另一个核心技术 Cgroups。那么究竟什么是Cgroups？我们应该如何使用 Cgroups？Docker 又是如何使用 Cgroups的？

#### Cgroups 定义

Cgroups（全称：control groups）是 Linux 内核的一个功能，它可以实现限制进程或者进程组的资源（如 CPU、内存、磁盘 I/O 等）。

#### Cgroups 功能及核心概念

Cgroups 主要提供了如下功能。

* 资源限制： 限制资源的使用量，例如我们可以通过限制某个业务的内存上限，从而保护主机其他业务的安全运行。
* 优先级控制：不同的组可以有不同的资源（ CPU 、磁盘 IO 等）使用优先级。
* 审计：计算控制组的资源使用情况。
* 控制：控制进程的挂起或恢复。

了解了 cgroups 可以为我们提供什么功能，下面我来看下 cgroups 是如何实现这些功能的。

cgroups 功能的实现依赖于三个核心概念：子系统、控制组、层级树。

* 子系统（subsystem）：是一个内核的组件，一个子系统代表一类资源调度控制器。例如内存子系统可以限制内存的使用量，CPU 子系统可以限制 CPU 的使用时间。
* 控制组（cgroup）：表示一组进程和一组带有参数的子系统的关联关系。例如，一个进程使用了 CPU 子系统来限制 CPU 的使用时间，则这个进程和 CPU 子系统的关联关系称为控制组。
* 层级树（hierarchy）：是由一系列的控制组按照树状结构排列组成的。这种排列方式可以使得控制组拥有父子关系，子控制组默认拥有父控制组的属性，也就是子控制组会继承于父控制组。比如，系统中定义了一个控制组 c1，限制了 CPU 可以使用 1 核，然后另外一个控制组 c2 想实现既限制 CPU 使用 1 核，同时限制内存使用 2G，那么 c2 就可以直接继承 c1，无须重复定义 CPU 限制。

cgroups 的三个核心概念中，子系统是最核心的概念，因为子系统是真正实现某类资源的限制的基础。

常用的 cgroups 子系统包括 cpu、memory、pids。

通过 mount 命令查看一下当前系统已经挂载的cgroups信息：

```
$ sudo mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_prio,net_cls)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuacct,cpu)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
```

#### Docker 是如何使用 cgroups 的

首先，创建并启动一个 nginx 容器，并限制内存为 1G。

```shell
docker run -it -m=1g nginx
```

然后，进入 cgroups 内存子系统的目录

```shell
# ls -l /sys/fs/cgroup/memory
total 0
-rw-r--r--.  1 root root 0 Sep  1 11:50 cgroup.clone_children
--w--w--w-.  1 root root 0 Sep  1 11:50 cgroup.event_control
-rw-r--r--.  1 root root 0 Sep  1 11:50 cgroup.procs
-r--r--r--.  1 root root 0 Sep  1 11:50 cgroup.sane_behavior
drwxr-xr-x.  3 root root 0 Sep  5 10:50 docker
... 省略部分输出
```

看到有一个 docker 目录，该目录是 Docker 在内存子系统下创建的。在 docker 目录下，有之前创建的 nginx 容器的 ID 的目录，进入该目录，查看该容器的 memory.limit_in_bytes 文件的内容。

```shell
# cat memory.limit_in_bytes
1073741824
```

可以看到内存限制值正好是 1G。

**Docker 创建容器时，Docker 会根据启动容器的参数，在对应的 cgroups 子系统下创建以容器 ID 为名称的目录, 然后根据容器启动时设置的资源限制参数, 修改对应的 cgroups 子系统资源限制文件, 从而达到资源限制的效果。**

其实 cgroups 不仅可以实现资源的限制，还可以为我们统计资源的使用情况，容器监控系统的数据来源也是 cgroups 提供的。

另外，请注意 cgroups 虽然可以实现资源的限制，但是不能保证资源的使用。例如，cgroups 限制某个容器最多使用 1 核 CPU，但不保证总是能使用到 1 核 CPU，当 CPU 资源发生竞争时，可能会导致实际使用的 CPU 资源产生竞争。

### 联合文件系统

联合文件系统（Union File System，Unionfs）是一种通过创建文件层进程操作的文件系统。是一种分层的轻量级文件系统，它可以把多个目录内容联合挂载到同一目录下，从而形成一个单一的文件系统，这种特性可以让使用者像是使用一个目录一样使用联合文件系统。

Docker 使用联合文件系统为容器提供构建层，使得容器可以实现写时复制以及镜像的分层构建和存储。常用的联合文件系统由 AUFS、Overlay 和Devicemapper等。

#### **AUFS**



#### **Devicemapper**

#### **OverlayFS**

### 结语

容器技术从 1979 年 chroot 的首次问世便已崭露头角，但是到了 2013 年，Dokcer 的横空出世才使得容器技术迅速发展。而Docker爆发的原因是，在同类产品的基础上加入了镜像功能，并且封装了镜像仓库使得镜像分发更加方便。

Docker 还提供了工具和平台来管理容器的生命周期：

1. 使用容器开发应用程序及其支持组件。
2. 容器成为分发和测试你的应用程序的单元。
3. 可以将应用程序作为容器或协调服务部署到生产环境中。无论您的生产环境是本地数据中心，云提供商还是两者的混合，其工作原理都相同。