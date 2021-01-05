## Docker命令

https://docs.docker.com/reference/

#### 帮助命令

```shell
docker version
docker info
docker --help
docker 命令 --help
```

#### 镜像命令

查看

```shell
docker images \
	-a, --all		# 列出所有镜像
	-q, --quiet		# 只显示镜像id
```

搜索

```shell
docker search mysql
	--filter=STARS=3000		# 搜索STARS大于3000的镜像
```

下载

```shell
docker pull mysql:5.7		# 下载镜像，不指定版本则默认下载latest
5.7: Pulling from library/mysql
852e50cd189d: Pull complete 	# layer分层下载，docker image的核心 # 联合文件系统
29969ddb0ffb: Pull complete 
a43f41a44c48: Pull complete 
5cdd802543a3: Pull complete 
b79b040de953: Pull complete 
938c64119969: Pull complete 
7689ec51a0d9: Pull complete 
36bd6224d58f: Pull complete 
cab9d3fa4c8c: Pull complete 
1b741e1c47de: Pull complete 
aac9d11987ac: Pull complete 
Digest: sha256:8e2004f9fe43df06c3030090f593021a5f283d028b5ed5765cc24236c2c4d88e	# 签名
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7		# 真实地址，docker pull mysql:5.7 等价于 docker pull docker.io/library/mysql:5.7
```

删除

```shell
docker rmi 镜像id/镜像名称		# 删除镜像
	-f		# 强制删除
	
docker rmi -f 镜像id1 镜像id2 镜像id3 	# 删除多个镜像

docker rmi -f $(docker images -aq)	# 删除全部镜像
```

#### 容器命令

新建容器并启动

```shell
docker run [可选参数] IMAGE
	--name="Name"	# 指定容器名称
	-d				# 后台方式运行
	-it				# 使用交互方式运行，进入容器查看内容
	-p				# 指定容器端口	-p 8080:8080
			# -p ip:主机端口:容器端口
			# -p 主机端口:容器端口
			# -p 容器端口
			# 容器端口
	-P				# 随机指定端口
	-e		# 环境变量
	--rm	# 用完后删除容器，一般用来测试
	
docker run -it mysql:5.7 /bin/bash		#启动并进入容器
root@77d26c33cc99:/#
```

查看

```shell
docker ps
	-a		# 查看全部，包括已退出的容器
	-n=3	# 显示最近创建的3个容器
	-q		# 只显示容器id
```

删除

```shell
docker rm 容器id	
	-f 		# 强制删除，包括正在运行的
docker rm -f $(docker ps -aq)		# 删除所有容器
docker ps -a -q | xargs docker rm	# 删除所有容器
```

退出容器交互模式

```shell
exit			# 直接退出
Ctrl + P + D	# 容器不停止退出
```

启动、停止

```shell
docker start 容器id		# 启动
docker restart 容器id		# 重启
docker stop 容器id		# 停止当前正在运行的容器
docker kill 容器id		# 强制停止当前容器
docker pause 容器id		# 暂停
docker unpause 容器id		# 从暂停状态重新运行
```

#### 其他命令

后台启动容器

```shell
docker run -d centos
docker ps	# 发现centos停止了

# 常见的坑，docker容器使用后台运行，必须要有一个前台进程，如果docker发现没有应用，就会自动停止
```

**查看日志**

```shell
docker logs 容器id
	-f				# 滚动查看
	-t				# 时间戳
	--tail 10		# 显示的记录数
```

**查看容器中的进程信息**

```shell
docker top 容器id
```

**查看镜像的元数据**

```shell
docker inspect 容器id
```

**进入当前正在运行的容器**

```shell
# 通常容器都是使用后台方式运行，需要进入容器，修改配置等

# 方式一	进入容器后开启一个新的终端，可以在里面操作
docker exec -it 容器id /bin/bash

# 方式二	进入容器正在执行的终端	todo
docker attach 容器id
```

从容器内拷贝文件到主机上

```shell
docker cp 容器id：容器内路径 主机路径
```

查看容器的cpu、内存占用

```shell
docker stats
```



