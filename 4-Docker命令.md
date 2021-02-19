## Docker命令

https://docs.docker.com/reference/

#### 帮助

```shell
docker --help
docker command --help
```

#### info|version

* info
* version

#### 镜像仓库

* login  登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

* logout 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

* pull

  ```shell
  # Registry 为注册服务器，默认为 docker.io，可替换为自己的注册服务器
  # Repository 为镜像仓库，通常把一组相关联的镜像归为一个镜像仓库，默认为 library
  # Image 为镜像名称
  # Tag 为镜像的标签，不指定则默认为 latese
  # 先从本地搜索，本地搜索不到则从远程仓库下载
  docker pull [Registry]/[Repository]/[Image]:[Tag]
  
  docker pull mysql:5.7
  	--disable-content-trust	# 忽略镜像的校验，默认为开启
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
  docker.io/library/mysql:5.7		# 真实地址
  ```

* push 将本地的镜像上传到镜像仓库，要先登陆到镜像仓库

* search 从Docker Hub查找镜像

  ```shell
  docker search mysql \
  	--filter=STARS=3000		# 搜索STARS大于3000的镜像
  ```

#### 本地镜像管理

* images 列出本地镜像

  ```shell
  docker images \
  	-a, --all		# 列出所有镜像
  	-q, --quiet		# 只显示镜像id
  ```

* rmi 删除本地一个或多少镜像

  ```shell
  docker rmi 镜像id/镜像名称
  	-f		# 强制删除
  	
  docker rmi -f 镜像id1 镜像id2 镜像id3 	# 删除多个镜像
  
  docker rmi -f $(docker images -aq)		# 删除全部镜像
  ```

* tag 重命名镜像，可将其归入某一仓库

  ```shell
  docker tag [OPTIONS] IMAGE:[Tag] [Registry]/[Repository]/[Image]:[Tag]
  
  docker tag ubuntu:15.10 zjt/ubuntu:v1
  ```

* build 使用 Dockerfile 构建镜像

  ```shell
  docker build 
  	-f		# 指定要使用的Dockerfile路径，不指定则默认当前文件夹的Dockerfile文件
  	-t		# 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签
  	
  docker build -f /path/to/a/Dockerfile -t runoob/ubuntu:v1 . 
  ```

* history 查看指定镜像的创建历史

* save 将指定镜像保存成 tar 归档文件

* load 导入使用 `docker save` 命令导出的镜像

* import 从容器`docker export`生成的归档文件中创建镜像

#### 容器生命周期管理

* run

  ```shell
  docker run [可选参数] IMAGE
  	--name="Name"	# 指定容器名称
  	-d				# 后台方式运行，并返回容器ID
  	-it				# 使用交互方式运行，为容器重新分配一个伪终端
  	-p				# 指定容器端口	-p 8080:8080
                      # -p ip:主机端口:容器端口
                      # -p 主机端口:容器端口
                      # -p 容器端口
                      # 容器端口
  	-P				# 随机端口映射，容器内部端口随机映射到主机的端口
  	-e				# 环境变量
  	--rm			# 用完后删除容器，一般用来测试
  	-v				# 绑定一个卷
  	--net			# 指定容器的网络连接类型，支持 bridge/host/none/container 四种类型
  	
  docker run -it mysql:5.7 /bin/bash		#启动并进入容器
  root@77d26c33cc99:/#
  
  # 退出容器交互模式
  exit			# 直接退出
  Ctrl + P + D	# 容器不停止退出
  ```

  后台启动容器

  ```shell
  docker run -d centos
  docker ps	# 发现centos停止了
  
  # 常见的坑，docker容器使用后台运行，必须要有一个前台进程，如果docker发现没有应用，就会自动停止
  ```

* start/stop/restart

* kill 强制停止

* rm 删除容器

  ```shell
  docker rm 容器id	
  	-f 		# 强制删除，包括正在运行的
  	-l		# 移除容器间的网络连接，而非容器本身
  	-v		# 删除与容器关联的卷
  docker rm -f $(docker ps -aq)		# 删除所有容器
  docker ps -a -q | xargs docker rm	# 删除所有容器
  ```

* pause/unpause

* create 创建一个新的容器但不启动它

* exec  在运行的容器中执行命令

  ```shell
  docker exec
  	-i		# 读取标准输入STDIN
  	-t		# 分配一个伪终端
  	-d		# 后台运行
  	
  docker exec -it centos /bin/bash
  ```

#### 容器操作

* ps 列出容器

  ```shell
  docker ps
  	-a		# 查看全部的容器，包括未运行
  	-n=3	# 显示最近创建的3个容器
  	-q		# 只显示容器id
  ```

  容器状态有7种：

  - created（已创建）
  - restarting（重启中）
  - running（运行中）
  - removing（迁移中）
  - paused（暂停）
  - exited（停止）
  - dead（死亡）

* inspect  查看容器/镜像的元数据

* top  查看容器中运行的进程信息

* stats  查看容器的cpu、内存占用

* attach  attach到一个已经运行的容器的stdin，然后进行命令执行的动作

  **注意：**CTRL-C不仅会退出容器，还会导致容器停止。可以带上 --sig-proxy=false 来确保CTRL-D或CTRL-C不会关闭容器。

* events 从服务器获取实时事件

* logs

  ```shell
  docker logs 容器id
  	-f				# 滚动查看
  	-t				# 时间戳
  	--tail 10		# 仅列出最新N条容器日志
  	--since			# 显示某个开始时间的所有日志
  ```

* wait 阻塞运行直到容器停止，然后打印出它的退出代码

* export 将文件系统作为一个tar归档文件导出到STDOUT

* port 列出指定的容器的端口映射

#### 容器rootfs命令

* commit 从运行中的容器提交为镜像

  ```shell
  docker commit 容器id
  	-a				# 提交的镜像作者
  	-c				# 使用Dockerfile指令来创建镜像
  	-m				# 提交时的说明文字
  	-p				# 在commit时，将容器暂停
  ```

* cp 用于容器与主机之间的数据拷贝

* diff 检查容器里文件结构的更改

  结果字段说明

  * A	添加了文件或目录
  * D	删除了文件或目录
  * C	修改了文件或目录

#### 其他

进入当前正在运行的容器，并在交互控制台执行命令（通常容器都是使用后台方式运行，需要进入容器操作）

```shell
# 方式一	退出不会导致容器停止，大部分时候用这种方式
docker exec -it 容器id /bin/bash

# 方式二	CTRL-C不仅会退出伪终端，还会导致容器停止。可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。
docker attach 容器id
```

