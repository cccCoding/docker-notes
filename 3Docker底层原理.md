## Docker底层原理

#### docker run 流程

```
docker run hello-world
```

1. 在本机寻找镜像
2. 有则使用这个镜像运行，没有则去 DockerHub 上下载
3. 运行镜像



#### 底层原理

Docker 是一个Client-Server 结构的系统，Docker 的守护进程运行在宿主机上，通过 Socket 从客户端访问。

DockerServer 接收到 Docker-Client的指令，执行命令。

架构图todo

**Docker为什么比VM快**

1. Docker有着比虚拟机更少的抽象层
2. Docker用的是宿主机的内核，VM需要Guest OS，不需要重新加载一个操作系统内核，避免引导。

