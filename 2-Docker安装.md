## Docker安装

https://docs.docker.com/engine/install/centos/

建议 CentOS7 及以上的发行版本。建议使用`overlay2`存储驱动程序。

### 安装

1. 卸载旧版本

   ```bash
   yum remove docker \
       docker-client \
       docker-client-latest \
       docker-common \
       docker-latest \
       docker-latest-logrotate \
       docker-logrotate \
       docker-engine
   ```

2. 安装工具包

   ```bash
   yum install -y yum-utils
   ```

3. 设置镜像仓库

   ```bash
   yum-config-manager \
       --add-repo \
       https://download.docker.com/linux/centos/docker-ce.repo
       
   #使用阿里云镜像仓库
   yum-config-manager \
       --add-repo \
       http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

4. 更新yum软件包索引

   ```bash
   yum makecache fast
   ```

5. 安装最新版本 Docker Engine-Community 和 containerd

   ```bash
   yum install docker-ce docker-ce-cli containerd.io
   ```

   查看所有仓库中的版本并安装指定版本

   ```bash
   yum list docker-ce --showduplicates | sort -r
   
   yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
   
   yum install docker-ce-18.09.1 docker-ce-cli-18.09.1 containerd.io
   ```

6. 启动docker

   ```bash
   systemctl start docker
   ```

7. 验证安装

   ```bash
   docker version
   
   docker run hello-world
   ```

   > 安装完成后默认 docker 命令只能以 root 用户执行，如果想允许普通用户执行 docker 命令，需要执行以下命令 sudo groupadd docker && sudo gpasswd -a ${USER} docker && sudo systemctl restart docker ，执行完命令后，退出当前命令行窗口并打开新的窗口即可。

8. 配置阿里云镜像加速

   阿里云，容器镜像服务，镜像加速器

   ```bash
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://t4rq807k.mirror.aliyuncs.com"]
   }
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

### 卸载

1. 卸载软件包

   ```bash
   yum remove docker-ce docker-ce-cli containerd.io
   ```

2. 删除资源 images, containers, and volumes

   ```bash
   # docker默认工作路径为 /var/lib/docker
   rm -rf /var/lib/docker
   ```
