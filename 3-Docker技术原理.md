# Docker 技术原理

Docker 是基于 Linux 的 Namespace 、Cgroups 和联合文件系统这三大核心技术实现的。使用 Namespace 做主机名、网络、PID 等资源的隔离，使用 Cgroups 对进程或者进程组做资源（例如：CPU、内存等）的限制，联合文件系统用于镜像构建和容器运行环境。

## chroot

chroot 是最早的容器雏形。

### **定义**

> chroot 是在 Unix 和 Linux 系统的一个操作，针对正在运作的软件进程和它的子进程，改变它外显的根目录。一个运行在这个环境下，经由 chroot 设置根目录的程序，它不能够对这个指定根目录之外的文件进行访问动作，不能读取，也不能更改它的内容。

通俗地讲，chroot 可以把任何目录更改为当前进程的根目录，使这个程序不能访问目录之外的其他目录。

## Namespace

Docker 是使用 Linux 的 Namespace 技术实现各种资源隔离的。

### 定义

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

虽然 Linux 内核提供了 8 种 Namespace，但是最新版本的 Docker 只使用了其中的前 6 种。

### 各种 Namespace 的作用

#### **（1）Mount Namespace**

用来隔离不同的进程或进程组看到的挂载点。通俗地说，就是可以实现在不同的进程中看到不同的挂载目录。使用 Mount Namespace 可以实现容器内只能看到自己的挂载信息，在容器内的挂载操作不会影响主机的挂载目录。

#### **（2）PID Namespace**

PID Namespace 的作用是用来隔离进程。在不同的 PID Namespace 中，进程可以拥有相同的 PID 号，利用 PID Namespace 可以实现每个容器的主进程为 1 号进程，而容器内的进程在主机上却拥有不同的PID。例如一个进程在主机上 PID 为 122，使用 PID Namespace 可以实现该进程在容器内看到的 PID 为 1。

#### **（3）UTS Namespace**

UTS Namespace 主要是用来隔离主机名的，它允许每个 UTS Namespace 拥有一个独立的主机名。例如我们的主机名称为 docker，使用 UTS Namespace 可以实现在容器内的主机名称为 zjt 或者其他任意自定义主机名。

#### **（4）IPC Namespace**

IPC Namespace 主要是用来隔离进程间通信的。例如 PID Namespace 和 IPC Namespace 一起使用可以实现同一 IPC Namespace 内的进程彼此可以通信，不同 IPC Namespace 的进程却不能通信。

#### **（5）User Namespace**

User Namespace 主要是用来隔离用户和用户组的。一个比较典型的应用场景就是在主机上以非 root 用户运行的进程可以在一个单独的 User Namespace 中映射成 root 用户。使用 User Namespace 可以实现进程在容器内拥有 root 权限，而在主机上却只是普通用户。

User Namesapce 的创建是可以不使用 root 权限的。

#### **（6）Net Namespace**

Net Namespace 是用来隔离网络设备、IP 地址和端口等信息的。Net Namespace 可以让每个进程拥有自己独立的 IP 地址，端口和网卡信息。例如主机 IP 地址为 172.16.4.1 ，容器内可以设置独立的 IP 地址为 192.168.1.1。

### 为什么 Docker 需要 Namespace

当 Docker 新建一个容器时， 它会创建这六种 Namespace，然后将容器中的进程加入这些 Namespace 之中，使得 Docker 容器中的进程只能看到当前 Namespace 中的系统资源。

正是由于 Docker 使用了 Linux 的这些 Namespace 技术，才实现了 Docker 容器的隔离。

## Cgroups

使用不同的 Namespace，可以实现容器中的进程看不到别的容器的资源，但是容器内的进程仍然可以任意地使用主机的 CPU 、内存等资源，如果某一个容器使用的主机资源过多，可能导致主机的资源竞争，进而影响业务。

需要用 Linux 内核的另一个核心技术 Cgroups 来限制一个容器资源的使用。

### 定义

Cgroups（control groups）是 Linux 内核的一个功能，它可以实现限制进程或者进程组的资源（如 CPU、内存、磁盘 I/O 等）。

### Cgroups 功能及核心概念

Cgroups 主要提供了如下功能。

* 资源限制： 限制资源的使用量，例如我们可以通过限制某个业务的内存上限，从而保护主机其他业务的安全运行。
* 优先级控制：不同的组可以有不同资源的使用优先级。
* 审计：计算控制组的资源使用情况。
* 控制：控制进程的挂起或恢复。

Cgroups 功能的实现依赖于三个核心概念：子系统、控制组、层级树。

* 子系统（subsystem）：是一个内核的组件，一个子系统代表一类资源调度控制器。例如内存子系统可以限制内存的使用量，CPU 子系统可以限制 CPU 的使用时间。
* 控制组（cgroup）：表示一组进程和一组带有参数的子系统的关联关系。例如，一个进程使用了 CPU 子系统来限制 CPU 的使用时间，则这个进程和 CPU 子系统的关联关系称为控制组。
* 层级树（hierarchy）：是由一系列的控制组按照树状结构排列组成的。这种排列方式可以使得控制组拥有父子关系，子控制组会继承于父控制组的属性。

Cgroups 的三个核心概念中，子系统是最核心的概念，因为子系统是真正实现某类资源的限制的基础。

常用的 cgroups 子系统包括 cpu、memory、pids。

通过 mount 命令查看一下当前系统已经挂载的 cgroups 信息：

```shell
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

### Docker 是如何使用 Cgroups 的

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

**Docker 创建容器时，Docker 会根据启动容器的参数，在对应的 cgroups 子系统下创建以容器 ID 为名称的目录，然后根据容器启动时设置的资源限制参数， 修改对应的 cgroups 子系统资源限制文件，从而达到资源限制的效果。**

**而且 cgroups 不仅可以实现资源的限制，还可以为我们统计资源的使用情况，容器监控系统的数据来源也是 cgroups 提供的。**

## 联合文件系统

联合文件系统（Union File System，Unionfs）是一种分层的轻量级文件系统，它可以把多个目录内容联合挂载到同一目录下，从而形成一个单一的文件系统，这种特性可以让使用者像是使用一个目录一样使用联合文件系统。

联合文件系统是 Docker 镜像和容器的基础。Docker 使用联合文件系统，实现镜像的分层构建和存储，多个业务基于同一基础镜像构建，基础镜像只需存储一份，从而节省大量存储空间；为容器提供构建层，使得容器可以实现写时复制。

常用的联合文件系统有 AUFS、Devicemapper 和 Overlay。

### **AUFS**

Docker 最早使用的常用联合文件系统。

#### 如何配置 Docker 的 AUFS 模式

AUFS 目前并未被合并到 Linux 内核主线，因此只有 Ubuntu 和 Debian 等少数操作系统支持 AUFS。你可以使用以下命令查看你的系统是否支持 AUFS：

```shell
$ grep aufs /proc/filesystems
nodev   aufs
```

如果输出结果包含aufs，则代表当前操作系统支持 AUFS。当确认完操作系统支持 AUFS 后，即可配置 Docker 的启动参数。

编辑 /etc/docker/daemon.json 文件，如果该文件不存在，则创建该文件，并添加以下配置：

```shell
{
  "storage-driver": "aufs"
}
```

重启 Docker：

```shell
$ sudo systemctl restart docker
```

查看是否生效：

```shell
$ sudo docker info
```

#### AUFS 工作原理

AUFS 是联合文件系统，意味着它在主机上使用多层目录存储，**每一个目录在 AUFS 中都叫作分支，而在 Docker 中则称之为层（layer），但最终呈现给用户的则是一个普通单层的文件系统，我们把多层以单一层的方式呈现出来的过程叫作联合挂载。**

![Lark20201014-171313.png](https://s0.lgstatic.com/i/image/M00/5E/82/CgqCHl-GwcCAOu4aAABzKSlpRlI180.png)

每一个镜像层和容器层都是 /var/lib/docker 下的一个子目录，镜像层和容器层都在 aufs/diff 目录下，每一层的目录名称是镜像或容器的 ID 值，联合挂载点在 aufs/mnt 目录下，mnt 目录是真正的容器工作目录。

#### AUFS 是如何工作的

AUFS 的工作过程中对文件的操作分为读取文件和修改文件。

**读取文件**

当我们在容器中读取文件时，可能会有以下场景。

* 文件在容器层中存在时：直接从容器层读取。

* 文件在容器层中不存在时：从镜像层查找该文件，然后读取文件内容。

* 文件既存在于镜像层，又存在于容器层：从容器层读取该文件。

**修改文件或目录**

AUFS 对文件的修改采用的是**写时复制**的工作机制，这种工作机制可以最大程度节省存储空间。

具体的文件操作机制如下。

* 第一次修改文件：当我们第一次在容器中修改某个文件时，AUFS 会触发写时复制操作，AUFS 首先从镜像层复制文件到容器层，然后再执行对应的修改操作。

> AUFS 写时复制的操作将会复制整个文件，如果文件过大，将会大大降低文件系统的性能，因此当我们有大量文件需要被修改时，AUFS 可能会出现明显的延迟。好在，写时复制操作只在第一次修改文件时触发，对日常使用没有太大影响。

* 删除文件或目录：当文件或目录被删除时，AUFS 并不会真正从镜像中删除它，因为镜像层是只读的，AUFS 会创建一个特殊的文件或文件夹，这种特殊的文件或文件夹会阻止容器的访问。

### **Devicemapper**

CentOS 系统通常使用 Devicemapper 作为 Docker 的联合文件系统。

Devicemapper 使用块设备来存储文件，运行速度会比直接操作文件系统更快，因此很长一段时间内在 Red Hat 或 CentOS 系统中，Devicemapper 一直作为 Docker 默认的联合文件系统驱动。

#### 定义

Devicemapper 是 Linux 内核提供的框架，从 Linux 内核 2.6.9 版本开始引入，Devicemapper 与 AUFS 不同，AUFS 是一种文件系统，而 Devicemapper 是一种映射块设备的技术框架。

#### 原理

Devicemapper 实现镜像分层的根本原理是快照。Docker 使用了 Devicemapper 瘦供给模块的快照（snapshot）技术。

### **OverlayFS**

在 Docker 中 OverlayFS 文件驱动被分为了两种，一种是早期的overlay，不推荐在生产环境中使用，另一种是更新和更稳定的 overlay2，推荐在生产环境中使用。

**overlay2 目前已经是 Docker 官方推荐的文件系统了，也是目前安装 Docker 时默认的文件系统。**overlay2 在生产环境中不仅有着较高的性能，它的稳定性也极其突出，因为 overlay2 在 inode 优化上更加高效。因此在生产环境中推荐使用 overlay2 作为 Docker 的文件驱动。

#### 使用 overlay2 的先决条件

overlay2 虽然很好，但是它的使用是有一定条件限制的。

* Docker 版本必须高于 17.06.02

* 如果你的操作系统是 RHEL 或 CentOS，Linux 内核版本必须使用 3.10.0-514 或者更高版本，其他 Linux 发行版的内核版本必须高于 4.0（例如 Ubuntu 或 Debian），你可以使用`uname -a`查看当前系统的内核版本。

* overlay2 最好搭配 xfs 文件系统使用，并且使用 xfs 作为底层文件系统时，d_type必须开启，可以使用以下命令验证 d_type 是否开启：

  ```shell
  $ xfs_info /var/lib/docker | grep ftype
  naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
  ```

  当输出结果中有 ftype=1 时，表示 d_type 已经开启。如果你的输出结果为 ftype=0，则需要重新格式化磁盘目录，命令如下：

  ```shell
  $ sudo mkfs.xfs -f -n ftype=1 /path/to/disk
  ```

  另外，在生产环境中，推荐挂载 /var/lib/docker 目录到单独的磁盘或者磁盘分区，这样可以避免该目录写满影响主机的文件写入，并且把挂载信息写入到 /etc/fstab，防止机器重启后挂载信息丢失。

  挂载配置中推荐开启 pquota，这样可以防止某个容器写文件溢出导致整个容器目录空间被占满。写入到 /etc/fstab 中的内容如下：

  ```shell
  $ UUID /var/lib/docker xfs defaults,pquota 0 0
  ```

  其中 UUID 为 /var/lib/docker 所在磁盘或者分区的 UUID 或者磁盘路径。

#### 如何在 Docker 中配置 overlay2

停止 Docker

```shell
$ sudo systemctl stop docker
```

备份 /var/lib/docker 目录

```shell
$ sudo cp -au /var/lib/docker /var/lib/docker.back
```

编辑 /etc/docker/daemon.json 文件，如果该文件不存在，则创建该文件，并添加以下配置：

```shell
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.size=20G",
    "overlay2.override_kernel_check=true"
  ]
}
```

其中 storage-driver 参数指定使用 overlay2 文件驱动，overlay2.size 参数表示限制每个容器根目录大小为 20G。限制每个容器的磁盘空间大小是通过 xfs 的 pquota 特性实现，overlay2.size 可以根据不同的生产环境来设置这个值的大小。在生产环境中开启此参数，可防止某个容器写入文件过大，导致整个 Docker 目录空间溢出。

启动 Docker：

```shell
$ sudo systemctl start docker
```

查看是否生效：

```shell
$ sudo docker info
```

可以看到 Storage Driver 已经变为 overlay2，并且 d_type 也是 true。

#### overlay2 工作原理

**overlay2 是如何存储文件的**

overlay2 和 AUFS 类似，它将所有目录称之为层（layer），overlay2 的目录是镜像和容器分层的基础，而把这些层统一展现到统一的目录下的过程称为联合挂载（union mount）。overlay2 把目录的下一层叫作`lowerdir`，上一层叫作`upperdir`，联合挂载后的结果叫作`merged`。

> overlay2 文件系统最多支持 128 个层数叠加，也就是说你的 Dockerfile 最多只能写 128 行，不过这在日常使用中足够了。

总体来说，overlay2 是这样储存文件的：overlay2 将镜像层和容器层都放在单独的目录，并且有唯一 ID，每一层仅存储发生变化的文件，最终使用联合挂载技术将容器层和镜像层的所有文件统一挂载到容器中，使得容器中看到完整的系统文件。

**overlay2 如何读取文件**

容器内进程读取文件分为以下三种情况。

* 文件在容器层中存在：当文件存在于容器层并且不存在于镜像层时，直接从容器层读取文件；

* 当文件在容器层中不存在：从镜像层查找该文件，然后读取文件内容；

* 文件既存在于镜像层，又存在于容器层：从容器层读取该文件。

**overlay2 如何修改文件或目录**

overlay2 对文件的修改采用的是写时复制的工作机制，这种工作机制可以最大程度节省存储空间。具体的文件操作机制如下。

* 第一次修改文件：当我们第一次在容器中修改某个文件时，overlay2 会触发写时复制操作，overlay2 首先从镜像层复制文件到容器层，然后在容器层执行对应的文件修改操作。

>  overlay2 写时复制的操作将会复制整个文件，如果文件过大，将会大大降低文件系统的性能，因此当我们有大量文件需要被修改时，overlay2 可能会出现明显的延迟。不过写时复制操作只在第一次修改文件时触发，对日常使用没有太大影响。

* 删除文件或目录：当文件或目录被删除时，overlay2 并不会真正从镜像中删除它，因为镜像层是只读的，overlay2 会创建一个特殊的文件或目录，这种特殊的文件或目录会阻止容器的访问。
