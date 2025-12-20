# 1 Docker 基础 

## 1 Docker 概述
### 1.1 虚拟机 vs 容器

**1、什么是虚拟机？**

+ 虚拟机（Virtual Machine）指通过<font style="color:#601BDE;">软件模拟的具有完整硬件系统功能的</font>、<font style="color:#601BDE;">运行在一个完全隔离环境中的完整计算机系统</font>。在实体计算机中能够完成的工作，在虚拟机中都能够实现。
+ 在计算机中创建虚拟机时，需要将实体机的部分硬盘和内存容量作为虚拟机的硬盘和内存容量。每个虚拟机都有独立的 CMOS、硬盘和操作系统，可以像使用实体机一样对虚拟机进行操作。虚拟机的代表产品：VMWare 和 OpenStack。

> **什么是 CMOS？**
>
> + **物理机中**，一块真实的、由电池供电的物理芯片，存储BIOS设置；
> + **虚拟机中**，由虚拟化软件模拟出来的一个软件组件，功能与物理CMOS相同。

**2、什么是容器？**

容器在操作系统层虚拟化，是一个标准的软件单元。容器是完全使用沙箱机制，相互之间不会有任何接口，并且容器性能开销极低，容器具备如下特性：

+ **随处运行**：容器可以将代码与配置文件和相关依赖库进行打包，从而确保在任何环境下的运行都是一致的。
+ **高资源利用率**：容器提供进程级的隔离，因此可以更加精细地设置 CPU 和内存的使用率，进而更好地利用服务器的计算资源。
+ **快速扩展**：每个容器都可作为单独的进程予以运行，并且可以共享底层操作系统的系统资源，这样一来可以加快容器的启动和停止效率。

**3、虚拟机和容器的区别与联系**

+ 虚拟机虽然可以隔离出很多「子电脑」，但占用空间更大，启动更慢；虚拟机软件可能需要付费，如VMWare。
+ 容器技术不需要虚拟出整个操作系统，只需要虚拟一个小规模的环境，类似「沙箱」。
+ 运行空间，虚拟机一般要几 GB 到 几十 GB 的空间，而容器只需要 MB 级甚至 KB 级。

| **特性** | **虚拟机** | **容器** |
| --- | --- | --- |
| 隔离级别 | 操作系统级 | 进程 |
| 隔离策略 | Hypervisor（虚拟机监控器） | Cgroups（控制组群） |
| 系统资源 | 5 ～ 15% | 0 ～ 5% |
| 启动时间 | 分钟级 | 秒级 |
| 镜像存储 | GB - TB | KB - MB |
| 集群规模 | 上百 | 上万 |
| 高可用策略 | 备份、容灾、迁移 | 弹性、负载、动态 |


### 1.2 Docker 简介
**1、什么是 Docker？**

+ Docker 是 Google 开源、采用 Go 语言实现的；基于 Linux 内核的 `cgroup`、`namespace` ，以及 OverlayFS类的 Union FS 等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术；
+ Docker 在容器基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等，提供简单易用的容器使用接口；由于隔离的进程独立于宿主和其它的隔离进程，因此也称其为容器。
+ Docker 将应用程序及其依赖，打包在一个镜像文件里。运行这个镜像文件，就会生成一个虚拟容器；程序在这个虚拟容器里运行，就像在真实的物理机上运行一样。

**2、为什么要用 Docker？**

+ 由于容器是进程级别的，和传统的虚拟机相比，具有许多优势如：启动快、资源占用少、体积小 、高效部署和交付等。

**3、Docker 主要用途？**

+ 提供一次性的环境。如：本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
+ 提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
+ 组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## 2 Docker 架构和核心概念
### 2.1 Docker 架构图
![](A1-01)![](.\images\A1-01.png)

简介如下：

+ `Client`：Docker 客户端，使用 Docker Api 与 Docker 的守护进程进行通信；
+ `docker machine`：一个简化 Docker 安装的命令行工具，如：VirtualBox、Microsoft Azure；
+ `Host`：Docker 主机，一个物理或者虚拟的机器，用于执行 Docker 守护进程和容器；
+ `Daemon`：Docker 守护进程；
+ `Container`：Docker 容器，独立运行的一个或一组应用；
+ `Images`：Docker 镜像，用于创建 Docker 容器的模板；
+ `Registry`：Docker 仓库，用来保存镜像。

### 2.2 镜像（Image）
操作系统分为内核和用户空间，于 Linux 而言，内核启动后，会挂 root 文件系统为其提供用户空间支持。<font style="color:#601BDE;"> 镜像（Image）是一个特殊的文件系统，就相当于一个 root 文件系统</font>。

Docker 镜像包含：运行时所需的<font style="color:#601BDE;">程序、库、资源、配置、参数</font>等<font style="color:#601BDE;">静态</font>文件，不包含任何动态数据，其内容在构建之后不会被改变。

**镜像分层存储：**

+ Docker 镜像不是像.iso 那样的打包文件，镜像是一个<font style="color:#601BDE;">逻辑概念</font>，<font style="color:#601BDE;">由多层文件系统联合组成</font>，是采用 `UnionFS` 技术设计的分层存储的架构。
+ Docker 镜像特点：
    - 构建镜像时，一层一层构建，前一层是后一层的基础；
    - 每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。

### 2.3 容器（Container）
镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，<font style="color:#601BDE;">镜像是静态的定义，容器是镜像运行时的实体</font>。容器可以被创建、启动、停止、删除、暂停等。

**容器的特点：**

+ 容器的本质是进程，容器进程运行在属于自己的独立的命名空间（区别于宿主中执行的进程）；
+ 因此容器可以拥有自己的 root 文件系统、网络配置、进程空间，甚至用户 ID 空间；
+ 容器内的进程是运行在一个隔离的环境里，使用起来像是在独立于宿主的系统下操作。

**容器存储层：**

+ 每一个容器运行时，以镜像为基础层，在其上创建一个为容器运行时读写而准备的<font style="color:#601BDE;">容器存储层</font>；
+ 容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡，并且缓存在容器存储层的数据也随之清空。
+ 按照 docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。
+ 所有文件写入操作，都应该使用数据卷（Volume）或绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高（不会丢失数据）。

### 2.4 仓库（Repository）
`Docker Registry`是<font style="color:#601BDE;">用于集中存储、分发镜像的服务</font>。

+ 一个 `Registry` 中可以包含多个`Repository`，每个仓库可以包含多个标签（`Tag`），每个标签对应一个镜像；
+ 通常，一个仓库包含同一软件不同版本的镜像，而标签常用于对应该软件的各个版本；
+ 可以通过格式形如：<font style="color:#601BDE;"><仓库名>:<标签></font> 来指定具体是哪个软件哪个版本的镜像。若没有标签，将以 `latest`作为默认标签，如：`ubuntu:16.04、ubuntu:latest`。

**Docker Registry 公开服务：**

Docker Registry 公开服务是开放给用户使用、允许用户管理镜像的	Registry 服务。一般公开服务允许用户免费上传、下载公开的镜像。常见的公开服务如：官方的 [Docker Hub](https://hub.docker.com/) 、[网易云镜像服务](https://c.163.com/hub#/m/library/) 、[阿里云镜像库](https://cr.console.aliyun.com/) 等。

**私有 Docker Registry**

用户还可以在本地搭建私有 Docker Registry。Docker 官方提供了 Docker Registry 镜像，可以直接使用做为私有 Registry 服务。

### 2.5 镜像、容器、仓库的关系
关系如下图：

![](A1-02)![](.\images\A1-02.png)

## 3 Docker 安装
我的目标是在阿里云 ECS 服务器上进行云原生环境整体练习，拆分事项有：

+ Docker
+ 博客搭建
+ Kubernetes
+ 微服务部署实战
+ CICD 流水线
+ 微服务部署进阶
+ 服务网格

所以先规划 ECS 整体目录。

### 3.1 ECS 整体目录
```latex
/
├── home/
│   └── your-username/           # 用户主目录
│       ├── projects/            # 项目代码
│       │   ├── blog/            # 博客源码
│       │   ├── cicd-scripts/    # CI/CD脚本
│       │   └── k8s-manifests/   # K8s配置清单
│       └── tools/               # 工具脚本
│
├── opt/                         # 主安装目录
│   ├── docker/                  # Docker相关
│   │   ├── data/                # Docker数据目录
│   │   └── config/              # Docker配置
│   ├── kubernetes/              # K8s相关
│   │   ├── bin/                 # K8s二进制文件
│   │   ├── config/              # K8s配置文件
│   │   └── addons/              # K8s插件
│   ├── gitlab/                  # GitLab（可选）
│   ├── jenkins/                 # Jenkins
│   │   ├── home/                # Jenkins主目录
│   │   └── plugins/             # Jenkins插件
│   └── monitoring/              # 监控组件
│
├── var/                         # 可变数据
│   ├── lib/
│   │   ├── docker/              # Docker默认数据目录（保持默认）
│   │   └── kubelet/             # Kubelet数据
│   ├── log/
│   │   ├── docker/              # Docker日志
│   │   ├── kubernetes/          # K8s组件日志
│   │   ├── jenkins/             # Jenkins日志
│   │   └── nginx/               # Nginx日志
│   └── backup/                  # 备份目录
│
└── data/                        # 应用数据（推荐挂载数据盘到这里）
    ├── docker-registry/         # 私有镜像仓库数据
    ├── gitlab-data/             # GitLab数据
    ├── mysql-data/              # 数据库数据
    ├── blog-data/               # 博客数据
    └── backup/                  # 数据备份
```

### 3.2 Ubuntu 上 Docker 安装与配置
```plain
# 1、创建目录
sudo mkdir -p /opt/docker/{data,config}
sudo mkdir -p /var/log/docker

# 2、安装 Docker（使用官方脚本）
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

## 3、查看版本
docker version;

# 4、配置 Docker 数据目录
sudo tee /etc/docker/daemon.json << EOF
{
  "data-root": "/opt/docker/data",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  },
  "registry-mirrors": [	
    "https://naljryvl.mirror.aliyuncs.com",      
    "https://docker.nju.edu.cn"
  ]
}
EOF

# 5、重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker

## 6、测试docker是否安装成功
docker pull hello-world;
docker run hello-world;
```

说明：

+ 创建目录，参考目录整体规划，其他目录可需要时再新增；
+ `tee`：作用类似于 T 型管道--从标准输入读取数据，输出到标准输出，并保存文件
    - <<EOF：here document，将两个 EOF 之间所有内容作为标准输入
    - tee 接收这个输入，同时：写入到 /etc/docker/daemon.json，并输出到屏幕（方便查看执行结果）
+ /etc 目录的作用：
    - 配置文件专用目录，系统和服务的主要配置文件都放这里
    - Docker 默认会到 /etc/docker/daemon.json 寻找配置
+ registry-mirrors：配置国内镜像加速器，
  - _注：https://naljryvl.mirror.aliyuncs.com 是阿里云服务的加速器，对阿里云服务有效_
  - _若 docker pull hello-world 无法拉取镜像，记得检查 DNS 解析_
    ```text
    > cat /etc/resolv.conf

    nameserver 8.8.8.8
    nameserver 114.114.114.114
    nameserver 223.5.5.5
    nameserver 127.0.0.53
    ```

## 1 使用镜像
### 1.1 获取镜像
使用 `docker pull` 命令从 Docker 镜像仓库获取镜像，格式为：

```latex
docker pull [选项] [Registry 地址[:端口号]/]仓库名[:标签]
```

示例：

```latex
# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a0bcbecc962e: Pull complete
a2abf6c4d29d: Pull complete
a9edb18cadd1: Pull complete
589b7251471a: Pull complete
186b1aaa4aa6: Pull complete
b4df32aa5a72: Pull complete
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

+ docker pull nginx ：没带 tag，则使用默认 tag；
+ 镜像是分层储存的，所以下载也是一层层下载；下载过程中给出了<font style="color:#601BDE;">每一层的 ID 的前 12 位</font>。
+ 下载结束后，给出该镜像完整的 sha256 的摘要，以确保下载一致性。

**运行nginx容器**日志如下：

```plain
root@iZwz9fcf5vfrvbtzhk81111:/data/docker-registry# docker run -it --rm nginx:latest bash
root@8856f0a79797:/# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

+ 从主服务器（root@iZwz9fcf5vfrvbtzhk81111），切换到容器（root@8856f0a79797） 说明执行成功了；
+ `docker run -it --rm nginx:latest bash`:
    - `-it` ：-i ，交互式操作，-t，终端，以 bash 命令方式交互；
    - `--rm` ：容器退出后随之删除，默认情况，退出容器不会立即删除，除非执行 `docker rm`
    - `nginx:latest` ：以 nginx:latest 镜像为基础启动容器
+ `cat /etc/os-release`：查看容器基本信息
+ 退出容器：exit，如果没退出则连续执行两次 exit

### 1.2 镜像操作
##### 列出镜像
+ `docker image ls`：列出已经下载的镜像，只显示顶层镜像；
+ `docker image ls -a`：列出包括中间层的所有镜像；
+ `docker image ls nginx` ：列出 nginx 镜像；
+ `docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"` ：格式化查看镜像；
+ `docker system df`：查看镜像、容器、数据卷所占用的空间。

##### 删除镜像
可使用 `docker image rm` 命令删除本地镜像，格式形如：`docker image rm [选项] <镜像1> [<镜像2> ...]`

+ <镜像> 可以是 <font style="color:#601BDE;">镜像短 ID、镜像长 ID、镜像名 或 镜像摘要</font>。
+ 查看镜像摘要：`docker image ls --digests`



**删除镜像的理解：**

+ 删除行为分为：<font style="color:rgb(80, 80, 80);background-color:rgba(127, 127, 127, 0.12);">Untagged</font> 、<font style="color:rgb(80, 80, 80);background-color:rgba(127, 127, 127, 0.12);">Deleted</font> ，因为一个镜像可能会有多个标签，只有所有标签都删除后，镜像才会删除

```plain
# 拉取不同标签的相同镜像
docker pull nginx:latest
docker pull nginx:1.21

# 删除 nginx:1.21
docker image rm nginx:1.21

# 输出，镜像层还被 nginx:latest 引用
-->Untagged: nginx:1.21

# 删除最后一个标签
docker image rm nginx:latest

# 输出 Untagged 、Deleted
-->
Untagged: nginx:latest
Deleted: sha256:2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073b41ef9727da6c851f
Deleted: sha256:e379e8aedd4d72bb4c529a4ca07a4e4d230b5a1d3f7a61bc80179e8f02421ad8
...
```

+ 镜像是多层存储结构，删除镜像是从上层向基础层方向依次进行判断删除。如果当前层没有任何层依赖，则可删除；如果有镜像依赖镜像的当前层，则不会触发删除该层的行为。
+ 若某镜像启动的容器存在（即便容器没运行），镜像也不可删除。

### 1.3 使用 docker commit 理解镜像构成
镜像是多层存储，每一层是在前一层的基础上进行修改；容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。

Docker 提供了 `docker commit` 命令，可以将容器的存储层保存下来成为镜像（<font style="color:#601BDE;">在原有镜像的基础上，叠加上容器的存储层，构成新的镜像</font>）。

语法：`docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]`

_注：使用 _`_docker commit_`_ 生成的镜像会添加大量的无关内容，使镜像变得臃肿；并且很难清除知道制作镜像过程中，执行过什么命令、怎么生成的镜，被称为__**黑箱镜像**__。_

_不使用 docker commit 定制镜像，定制镜像应该使用 Dockerfile 来完成。_

### 1.4 使用 Dockerfile 定制镜像
镜像的定制实际上是<font style="color:#601BDE;">定制每一层所添加的配置、文件</font>。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么上一节提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

在一个空白目录中，建立一个文本文件，并命名为 Dockerfile：

```plain
mkdir mynginx
cd mynginx
vim Dockerfile
```

Dockerfile 文件类容：

```plain
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

两个命令：

+ `FROM`：指定基础镜像
+ `RUN`：用来执行命令行命令，两种格式：
    - shell 格式：`RUN <命令>`
    - exec 格式：`RUN ["可执行文件", "参数1", "参数2"]`

##### 构建镜像
使用 `docker build` 命令进行镜像构建，格式为：

```latex
docker build [选项] <上下文路径/URL/->
```

当构建镜像时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

`docker build` 的工作原理：Docker 在运行时分为 Docker 引擎（服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 [Docker Remote API](https://docs.docker.com/develop/sdk/)，而如 `docker` 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 `docker` 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。

### 1.5 实现原理
每个镜像都由很多层次构成，Docker 使用 **Union FS** 将这些不同的层结合到一个镜像中去。

通常 Union FS 有两个用途：可以实现不借助 LVM、RAID 将多个 disk 挂到同一个目录下；将一个只读的分支和一个可写的分支联合在一起，Live CD 正是基于此方法可以在镜像不变的基础上允许用户在其上进行一些写操作。

Docker 在 OverlayFS 上构建的容器也是利用了类似的原理。

## 2 Dockerfile 指令详解
Dockerfile 指令除了上述介绍的 `FROM、RUN`，还有如下指令：

+ **COPY 复制文件**

```tcl
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```

+ **ADD 更高级的复制文件**  
和 COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。在 Docker 官方的 Dockerfile 最佳实践文档中要求，尽可能的使用 COPY，因为 COPY 的语义很明确，就是复制文件而已，而 ADD 则包含了更复杂的功能，其行为也不一定很清晰。
+ **CMD 容器启动命令**

```plain
shell 格式：CMD <命令>
exec 格式：CMD ["可执行文件", "参数1", "参数2"...]
```

参数列表格式：CMD ["参数1", "参数2"...]。在指定了 `ENTRYPOINT` 指令后，用 CMD 指定具体的参数。容器是进程，在容器启动时，需指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令，如：`docker run -it ubuntu cat /etc/os-release`。

+ **ENTRYPOINT 入口点**  
ENTRYPOINT 的格式和 RUN 指令格式一样，分为 exec 格式和 shell 格式。ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 `--entrypoin`t 来指定。
+ **ENV 设置环境变量**

```latex
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

## 3 操作容器
容器是独立运行的一个或一组应用，以及它们的运行态环境。

### 3.1 启动
启动容器有两种方式：

+ 基于镜像新建一个容器并启动，使用 `docker run`

```plain
docker run ubuntu:18.04 /bin/echo 'Hello world'
```

+ 将在终止状态（stopped）的容器重新启动，使用 `docker container start`

**守护态运行：**

可以添加 -d 参数，让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。如：

```plain
docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

**进入容器：**

当容器使用 -d 参数进入后台后，可以通过 `docker exec` 命令进入容器。

```plain
[root@izwz98jvb8bcz3rf5ydvzpz mynginx]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
51fdf30cc6c4        nginx:v2            "/docker-entrypoint.…"   12 seconds ago      Up 12 seconds              80/tcp              suspicious_morse
5d3b8ccf0bb4        hello-world         "/hello"                 4 hours ago         Exited (0) 6 minutes ago                       reverent_jepsen
[root@izwz98jvb8bcz3rf5ydvzpz mynginx]# docker exec -it  51fdf30cc6c4 bash
```

### 3.2 终止
使用 `docker container stop` 来终止一个运行中的容器。也可使用 `exit` 或 `ctrl+d` 来退出终端时，所创建的容器立刻终止。

```latex
D:\docker\mynginx> docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
b3a59b5dd631        nginx:v2            "/docker-entrypoint.…"   2 hours ago         Up 5 minutes        0.0.0.0:81->80/tcp   web2
D:\docker\mynginx> docker container stop b3a59b5dd631
```

### 3.3 导出和导入
使用 `docker export、docker import` 导出和导入容器快照。

### 3.4 删除
使用 `docker container rm` 来删除一个处于终止状态的容器。使用 `docker container prune` 清理所有处于终止状态的容器。

### 3.3 docker 常用命令
使用 `docker --help` 查看 docker 命令，可分为如下几类操作：

+ docker 操作：
    - 版本/信息：`docker [info | version]`
+ 容器操作 ：
    - 容器生命周期：`docker [ run | start | stop | restart | kill | rm | pause | unpause ]`
    - 容器运维：`docker [ ps | inspect | exec | logs | export | import | port ]`
    - 容器 rootfs ：`docker [ commit | cp | diff ]`
+ 镜像操作：
    - 镜像管理：`docker [ images | rmi | tag | build | history | save | import ]`
+ 仓库操作
    - 镜像仓库：`docker [ login | pull | push | search ]`

## 参考资料
+ [Docker 官方文档](https://docs.docker.com/)
+ [Docker — 从入门到实践](https://vuepress.mirror.docker-practice.com/)


