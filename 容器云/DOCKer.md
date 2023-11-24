[TOC]

# DOCKer

## 什么是docker？

#### Docker概述

Docker是基于Go 语言开发的开源项目

Docker使用容器化技术，有别于虚拟机技术，Docker不是一个完整的操作系统

Docker将应用程序和其依赖项打包到一个可移植的容器中。通过使用Docker，开发者可以在不同的环境中快速部署、交付和运行应用程序，而无需担心环境差异和依赖问题。

Docker帮助开发者简化应用程序的部署、提高开发效率和资源利用率，并促进了DevOps实践（运维与开发的合作）。

##### 比较Docker相对于虚拟机的优势

* 传统虚拟机，虚拟出一条硬件，运行一个完整的操作系统，然后在这个系统上运行软件
* 而Docker直接运行在宿主机内核里，容器自己是没有内核的，也没有虚拟硬件，所以就轻便许多了
* 每个容器间是互相隔离的，每个容器内都有自己的文件系统。互不影响
* 总的来说，轻便，快捷，安全，节省资源，可快速部署，极大地简化了应用程序的部署、管理和运维过程，并提供了更好的环境隔离和安全性

> DevOps(开发&运维)

**让应用更快速的交付和部署**

传统：需要写帮助文档，安装一大堆程序

Docker：打包镜像，发布测试，一键运行

**更便捷的升级和扩缩容**

项目打包为一个镜像，从服务器A移植到到服务器B（扩展）

**更简单的系统运维**

在容器化之后，我们的开发，测试环境都是高度一致的

**更高效的计算机资源利用**

Docker是内核级别的虚拟化，可以在一个电脑上运行很多的容器实例！

可以将服务器的性能压榨到极致。

##### Docker的名词解释

> **Clien**
>
> 客户端 
>
> 其功能有：
>
> docker bulid  构建容器
>
> docker pull  拉取容器
>
> docker run  运行容器



> **Docker_host**
>
> 服务器
>
> 其功能有：
>
> docker daemon  守护进程是Docker的核心组件，运行在本地机器或服务器里，负责监听Docker客户端发送的命令和请求，并执行相应的操作，如**创建、启动、停止、删除容器**等
>
> **images**  镜像，是构建和运行容器的基础，包含了一个操作系统环境和应用程序的**完整文件系统**。
>
> **containers**  容器，基于镜像创建的实例，一个容器可以被视为一个独立的、轻量级的**虚拟运行环境**，它具有自己的文件系统、进程空间和网络接口



> **Registry**
>
> 仓库（注册表）
>
> 是Docker镜像**存储和分发**的中央仓库。它是用于存储、管理和共享Docker镜像的服务器端应用程序
>
> 仓库分为公有和私有仓库
>
> Docker hub （国外，公有）
>
> 阿里云（ACR），华为云，网易云的容器服务器（国内，公有）
>
> 常见的与Registry相关的Docker命令包括：
>
> - docker pull：从Registry拉取镜像到本地。
> - docker push：将本地的镜像推送到Registry。
> - docker search：在Registry中搜索可用的镜像。
> - docker tag：给本地镜像打标签，以便上传到Registry。



> **Client-Server**
>
> **客户端与服务端**，客户端可以与Docker服务端进行通信，发送命令并获取响应。这种架构使得在不同的操作系统或远程计算机上管理和运行容器变得可能，同时提供了更灵活、可扩展的基础架构。



> **Socket**
>
> 是一种在计算机网络中用于实现**进程间通信的编程接口**。Socket允许不同的计算机上的进程通过网络进行通信，可以在客户端和服务器之间传输数据。



> **TAG**
>
> 在 Docker 中，镜像的 tag 是用来对不同版本或不同配置的镜像进行标记和区分的一个标识符。一个镜像可以有多个不同的 tag。
>
> 通常，Docker 镜像的 tag 由两部分组成：仓库和标签。仓库指定了镜像所属的命名空间或组织，而标签用来标识镜像的版本或特定配置。
>
> 例如，假设有一个名为 “myapp” 的镜像，你可以为它创建不同的标签来表示不同的版本或不同的配置，比如 “myapp:latest”、“myapp:v1.0”、“myapp:development” 等。



> **stars**
>
> “stars”（星标）是用来表示对一个项目或一个镜像的喜爱和关注程度的指标。当用户认为某个项目或镜像很有价值、有趣或有帮助时，他们可以选择给项目或镜像添加星标。



> **images information**
>
> REPOSITORY  #镜像的存储库   
> TAG                  #镜像的标签
> IMAGE ID        #镜像id
> CREATED        #创建时间
> SIZE                  #大小

> **latest**
>
> 镜像在docker hub 中的默认TAG，即最新版

> **$()**
>
> 是shell脚本中命令替换的语法。它允许将命令的输出结果作为参数传递给另一个命令或赋值给变量。
>
> 例如 docker rmi -f $(docker images -aq)

​	**运行时**

> 运行时（Runtime）是指在计算机系统中，用于执行程序的软件组件或环境。它负责解释和执行源代码，并提供程序所需的运行环境和支持。
>
> 在不同的上下文中，运行时可以有不同的含义。以下是几个常见的运行时：
>
> 1. **程序运行时（Program Runtime）**：指程序在执行期间所需的运行环境。例如，Java程序需要Java运行时环境（JRE）来解释执行Java字节码。
> 2. **操作系统运行时（Operating System Runtime）**：指操作系统提供的软件环境和服务，用于执行和管理应用程序。操作系统运行时负责管理进程、内存、文件系统、网络等资源，以及提供调度和通信机制。
> 3. **容器运行时（Container Runtime）**：指用于运行容器的软件组件。容器运行时负责创建和管理容器，包括启动和停止容器、配置网络和存储等，并在容器内部提供隔离和资源管理。
> 4. **JavaScript运行时（JavaScript Runtime）**：指用于在浏览器或服务器上执行JavaScript代码的环境。常见的JavaScript运行时包括Node.js和浏览器的JavaScript引擎。

## 安装Docker

```bash
#确定centos版本 3.1 64位
cat /etc/centos-release
#删除旧docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
#下载阿里云docker的yum源
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#安装docker
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#启动docker
systemctl start docker

#hello-world
docker run hello-world

#配置Docker镜像加速
[root@docker ~] mkdir -p /etc/docker
登录阿里云 拉取自己账号的加速地址 如下
[root@docker ~] sudo tee /etc/docker/daemon.json <<-'EOF'
> {
>   "registry-mirrors": ["https://088klgwj.mirror.aliyuncs.com"]
> }
> EOF
[root@docker ~] sudo systemctl daemon-reload
[root@docker ~] sudo systemctl restart docker

#启动docker
[root@docker ~] systemctl start docker

#查看版本信息
[root@docker ~] docker version

#跑hello-world
[root@docker ~] docker run hello-world

#从阿里云的仓库中拉取了一个hello-world镜像

#卸载依赖
yum remove docker-ce docker-ce-li contrainerd.io

#删除资源
rm -rf /var/lib/docker

```

## 使用Docker

#### run的流程

```bash
docker run hello-world

#按下回车以后，发生了什么？

按下回车后，Docker会在本地寻找镜像——如果找到了则使用这个镜像——如果在本地没有找到则连接Docker hub (我们是阿里云)去他们的仓库里寻找这个镜像——如果找到了则使用这个镜像——如果依然没找到则报错——使用镜像创建一个hello-world的容器，这个容器将执行hello-world种指定的命令，即输出"Hello from Docker!"
```

![image-20230705195744487](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230705195744487.png)

#### 底层原理

##### Docker是怎么工作的？

Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问

```bash
#以Docker run mysql 为例

DockerServer接受到Docker-Client的指令，该指令会被发送到守护进程中去，守护进程会查看在本地是否有对应的(mysql)镜像，如果没有则先在仓库（如Docker hub）中下载镜像，然后根据镜像创建容器（每个容器具有唯一的id），创建完成后守护进程会启动容器，容器里面运行着mysql服务。当Docker-Client执行某个命令时，实际上是在客户机的操作系统中执行的，Docker会将这个命令传递给mysql容器并将容器内的响应返回给客户机


```

![Untitled](D:\myPicture\Cloud\Untitled.png)

#### Docker的常用命令

##### 帮助命令

> ```bash
> docker version    #显示docker的版本信息
> 
> docker info       #显示docker的详细信息，包括镜像和容器的数量
> 
> docker 某命令 --help #帮助命令
> 
> 帮助文档地址：https://dockerdocs.cn/engine/reference/commandline/info/
> 
> 
> ```
>
> 

##### 镜像命令

###### 	显示镜像

> ```bash
> docker images    #列出所有镜像(默认隐藏中间镜像)，显示他们的存储库，标签以及大小
> docker images -a #显示所有镜像，包括中间镜像
> docker images -q #只显示镜像id
> ```

###### 	搜索镜像

> ```bash
> docker search   #在docker hub 中搜索镜像
> docker search -f stars=500 mysql  #搜索星标至少为500的与mysql相关的镜像
> ```

###### 	拉取镜像

> ```bash
> docker pull     #在docker hub 中拉取镜像
> docker pull deiban #在docker hub 中拉取默认TAG的deiban镜像，即:latest
> docker pull debian:jessie  #在docker hub 中拉取TAG为jessie的镜像
> 
> 以拉取mysql为例
> [root@docker ~] docker pull mysql
> Using default tag: latest  #如果不写TAG 则默认为latest
> latest: Pulling from library/mysql
> 72a69066d2fe: Pull complete #分层下载，docker image的核心 联合文件系统
> 93619dbc5b36: Pull complete 
> 99da31dd6142: Pull complete 
> 626033c43d70: Pull complete 
> 37d5d7efb64e: Pull complete 
> ac563158d721: Pull complete 
> d2ba16033dad: Pull complete 
> 688ba7d5c01a: Pull complete 
> 00e060b6d11d: Pull complete 
> 1c04857f594f: Pull complete 
> 4d7cfa90e6ea: Pull complete 
> e0431212d27d: Pull complete 
> Digest: sha256:e9027fe4d91c0153429607251656806cc784e914937271037f7738bd5b8e7709
> Status: Downloaded newer image for mysql:latest
> docker.io/library/mysql:latest #真实地址，防伪信息
> 
> docker pull mysql
>   #等价于
> docker pull docker.io/library/mysql:latest
> 
> [root@docker ~] docker pull mysql:5.7
> 5.7: Pulling from library/mysql
> 72a69066d2fe: Already exists #
> 93619dbc5b36: Already exists #
> 99da31dd6142: Already exists #
> 626033c43d70: Already exists #
> 37d5d7efb64e: Already exists #
> ac563158d721: Already exists #
> d2ba16033dad: Already exists #注意这里的分层下载，因为前面已经下载过一个版本，所以这里与他共用了一些文件，极大的节省了存储空间
> 0ceb82207cd7: Pull complete 
> 37f2405cae96: Pull complete 
> e2482e017e53: Pull complete 
> 70deed891d42: Pull complete 
> Digest: sha256:f2ad209efe9c67104167fc609cca6973c8422939491c9345270175a300419f94
> Status: Downloaded newer image for mysql:5.7
> docker.io/library/mysql:5.7
> ```

###### 	删除镜像

> ```bash
> docker rmi #删除镜像，i就是image的意思
> docker rmi -f c20987f18b13 #强制删除id为c20987f18b13的镜像
> docker rmi -f $(docker images -aq)  #批量删除所有镜像，docker images -aq是列出所有镜像的id，$()是shell脚本中命令替换的语法
> 
> ```
>
> 

##### 容器命令

###### 	新建容器并启动

> ```bash
> docker run [可选参数] image
> 
> #参数说明
> --name name 容器名字如tomcat01 tomcat02 用来区分
> 如 docker run --name 123 alpine 以alpine镜像启动了一个名为123的容器
> 
> -d				以后台方式运行
> -it       		 使用交互方式运行，比如进入容器查看内容
> -p                指定容器的端口 如 -p 8808
> 	-p ip:主机端口
> 	-p 主机端口:容器端口(常用)
> -P 				 随机指定端口 #注意大写P  
> 
> #举例
> docker run -it centos ls -a
> 
> 启动一个新的以centos这个镜像为基础的容器 并在容器内执行 ls -a 命令 然后返回到客户机上
> 
> docker run -it centos /bin/bash 
> 
> 将启动一个新的交互式的 CentOS 容器，并在容器中打开一个 Bash Shell，可以在其中执行命令和操作 CentOS 环境。
> 
> 
> ```

###### 	退出容器

> ```bash
> exit  退出容器并停止运行
> Ctrl+p+q  退出容器但不停止
> ```
>
> ​	

###### 	列出所有运行中的容器

> ```bash
> docker ps  列出正在运行的容器
> 
> #可选选项
> -a 列出当前正在运行的和历史运行的所有容器
> -n=? 列出最近创建的x个容器 -n=1 就是列出最近创建一个容器
> -q  只显示容器的id
> ```

###### 	删除容器

> ```bash
> docker rm 容器id
> docker rm -f $(docker ps -aq)  强制删除所有容器,-f是强制删除的选项
> ```

###### 	启动和停止容器

> ```bash
> docker start 容器id     #启动容器
> docker restart 容器id   #重启容器
> dcoker pause 容器id	  #暂停容器
> docker stop   容器id    #停止运行容器
> docker kill   容器id    #强制停止容器
> ```
>
> - `docker pause` 命令用于暂停容器的所有进程。暂停后，容器中的进程将停止执行，但**容器的状态仍保持在暂停状态，资源被冻结**。暂停后的容器可以通过`docker unpause`命令恢复运行。这个命令主要用于临时停止容器的运行，例如在容器占用过多资源时，使其暂时停止运行以释放资源。
> - `docker stop`命令用于停止容器。停止时，Docker会向容器发送一个`SIGTERM`信号，让容器内的进程有机会进行清理和关闭。如果**在规定时间内容器没有正常停止，Docker会发送一个`SIGKILL`信号终止容器**。停止后的容器可以使用`docker start`命令重新启动。
> - `docker kill`命令用于强制终止容器。与`docker stop`不同，**`docker kill`命令会直接发送`SIGKILL`信号给容器，强制终止容器内的进程**。这个命令通常用于立即停止容器运行，对于无法正常响应`SIGTERM`信号的容器，这是一种有效的终止方式。

###### 一点心得

实测只有在run完一个容器后,Ctrl+p+q退出但不停止,这个容器才在当前运行,可以被stop 、restart、 kill

##### 常用的其他命令

###### 	后台启动容器

> ```bash
> docker run -d 镜像名
> #这时docker ps 发现没有运行中的容器，因为虽然启动了容器但没有启动一个前台进程“-it”，所以docker发现自己没有提供服务，就会立刻停止运行。
> ```

###### 	查看日志

> ```bash
> docker logs -tf --tail 输出的行数 容器id/容器名
> 
> -f 跟踪日志输出
> -t 显示时间戳
> 
> #编写一个简单的shell脚本并持续打印
> docker run -d --name test centos /bin/sh -c "while true;do echo '521';sleep 1;done"  #将在后台每隔一秒打印一个521字符串
> 
> docker logs -tf --tail 8 test 
> 查询test容器的最近8行运行日志
> 
> docker logs --since=$(( $(date +%s) - 2 )) test
> 
> $(( $(date +%s) - 2 )) #返回当前时间-2s的相对时间
> 
> 
> ```

###### 	查看容器中的进程信息

> ```bash
> dcoker top 容器id
> 
> 如：
> [root@docker ~]# docker top 63dbc1d3dc3d
> UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
> root                1907                1888                1                   03:51               ?                   00:00:00            /bin/sh -c while true;do echo '521';sleep 1;done
> root                1930                1907                0                   03:51               ?                   00:00:00            /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
> 
> #名词解释
> UID  用户ID 
> PID  进程ID
> PPTD 父进程的ID
> C    CPU使用率，表示进程当前使用的CPU时间片的百分比。
> STIME 进程启动的时间
> TTY	 进程所在的终端设备，对于没有终端的进程，显示为问号（?）。
> TIME 进程的累计CPU使用时间
> CMD  进程的命令行 指执行该进程的完整命令
> 
> 
> ```
> 

###### 	查看镜像的元数据

> ```bash
> docker inspect 镜像id
> 
> 可以查看镜像的所有元数据，包括标签、创建时间、大小、层信息等。
> 
> #示例
> 
> [root@docker ~]docker inspect $(docker images -q | head -n 1)
> 将展示第一个镜像的详细信息
> ```

###### 	进入当前正在运行的容器

> ```bash
> 方法一：docker exec -it 容器id 
> 
> #示例
> 
> docker exec -it test /bin/bash
> 
> 进入test容器并使用bash命令解释器 （可以开始写代码了）
> 
> 方法二：docker attach 容器id
> 
> #区别
> docker exec  #进入容器后开启一个新的终端，可以在里面操作（常用）
> docker attach #进入容器正在运行的终端，不会启动新的进程
> ```
>

###### 	从容器内拷贝文件到主机上

> ```bash
> docker cp 容器id：容器内路径  目的主机路径
> 
> #举例
> [root@51fc4f9abfb5 /]# touch txt
> [root@51fc4f9abfb5 /]# vi txt
> [root@51fc4f9abfb5 /]# [root@docker ~]# 
> [root@docker ~]# docker cp text:/txt /666
> Successfully copied 2.05kB to /666
> [root@docker ~]# cat /666
> 1233
> 
> 成功将容器text里的/txt文件复制到主机的/666里面
> 
> #拷贝是一个手动过程，未来我们使用 -v 卷的技术，可以实现自动同步
> 
> ```

###### 命令小结

目前为止已经掌握了一小部分常见的docker命令 还有很多命令会在以后的过程中逐渐掌握

###### 练习一：部署nginx

```bash
docker search nginx

dokcer pull nginx

docker run -d --name nginx01 -p 8080:80 nginx
#运行nginx容器并设置主机端口为8080映射端口为80，可以通过http://主机ip/8080访问nginx
#本地访问 curl localhost:8080
补充端口映射的概念：
8080:80 "8080是你所指定的主机上的一个可用的端口，80是容器的一个端口，通过8080:80的形式，将80端口映射在8080上，使得外界可以通过8主机的端口8080访问容器，这就是端口映射"
#需要注意的是，确保所使用的主机端口是可用的且未被其他进程占用的，以免引起冲突。同时，还需要了解容器内服务实际监听的端口，以正确指定端口映射关系

docker exec -it nginx01 /bin/bash

docker stop nginx01
```

###### 练习二：部署Tomcat

```bash
docker run -it --rm tomcat:9.0
#--rm 是用完即删的选项 适合用于测试，官方推荐

docker pull tomact

docker run -d -p 3355:8800 --name tomcat01 tomcat

docker exec -it tomcat01 /bin/bash 
#进入容器内部以bash环境操作

root@e34d1ac188f4:/usr/local/tomcat# cd webapps
root@e34d1ac188f4:/usr/local/tomcat/webapps# ls
root@e34d1ac188f4:/usr/local/tomcat/webapps# 

#发现webapps目录下为空，所以判断这个镜像不完整，所以无法通过外部访问(404)

思路：将webapps.list复制到webapps下 补充完整

补充：docker stats 可以查看cpu信息
```

##### 可视化

* portainer(暂时先用)       Rancher(CI/CD再用)

  ```bash
  docker run -d -p 8088:9000 \
  --restart=always -v /var/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer'
  
  通过主机ip/8088来访问
  
  ```

#### Docker镜像详解

###### 	镜像是什么？

镜像是一种轻量级，可执行的软件包，其中打包了软件以及软件运行所需要的环境，包括代码、运行时、库、环境变量和配置文件

###### 	如何得到镜像

* 从仓库中拉取（阿里云，docker hub）
* 他人拷贝
* 自己制作

###### 	Dokcer镜像加载原理

* UnionFS(联合文件系统)

> 是一种分层，轻量级并且高性能的文件系统。它使用分层的文件系统结构，**将多个独立的文件系统合并成一个虚拟的文件系统**，并提供了在这个虚拟文件系统上进行读写操作的能力。
>
> 联合文件系统的主要特点包括：
>
> 1. **分层结构**：联合文件系统由多个文件系统层次结构组成。每个层次结构称为一个分支（Branch），而最终的虚拟文件系统则是这些分支的组合。
> 2. **写时复制**：当对联合文件系统进行写操作时，通常会使用写时复制（Copy-on-Write）的方式。这意味着**只有发生修改的部分会被写入新的位置**，而不是在原来的文件系统上进行直接修改。例如docker中拉取和删除镜像时
> 3. **层次透明**：联合文件系统将各个分支的文件系统层次结构合并成一个整体，并提供了层次透明的访问方式。用户可以像访问单个文件系统一样访问联合文件系统，而不需要关心底层的分支结构。

* Docker镜像加载原理

> Docker的镜像实际由一层一层的文件系统组成，这种层级文件系统叫联合文件系统（UnionFS）
>
> bootfs(boot file system)<引导文件系统>主要包含bootloader（加载程序）和kernel（内核），bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加速器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。
>
> rootfs(root file system)<根目录文件系统>包含操作系统的核心文件和目录，典型的有Linux系统中的/dev，/proc，/bin，/etc等标准目录和文件
>
> **平时我们安装的Centos都是好几个G，为什么Docker提供的才200M？**
>
> 因为对于一个精简的OS，rootfs可以做到很小，只需要包含基本的命令，工具和程序库就可以了，底层直接用host的kernel，自己只需要提供rootfs就饿可以了。由此可见对于不同的Linux发行版，bootfs是基本一致的，而rootfs会有差别，因此不同的发行版可以共用bootfs。
>
> **总结：一个精简的操作系统镜像只需要包含基本的命令、工具和程序库，因为底层的内核和其他系统组件由宿主机的操作系统提供。因此，根文件系统（rootfs）可以被大幅度地精简，只包含运行容器所需的最小依赖。这样就实现了镜像的小巧化。**



###### 分层理解

> 分层的镜像
>
> 1. **分层的构建**：Docker镜像的每个分层都包含了对文件系统的修改。具体来说，每个分层可以增加、删除或修改文件、目录和系统配置等。**每个分层的修改操作都会与上一层的文件系统进行对比，只记录新增或修改的部分，而不会重复存储相同文件**。这种分层的构建方式有效地减小了镜像的体积。
> 2. **分层的联合挂载**：在运行容器时，Docker使用联合挂载（Union Mount）的方式将镜像的各个分层叠加在一起。这样，不同层次的文件和目录都可以在容器的文件系统中被访问到。分层的联合挂载使得容器可以享受到镜像中所有分层的功能和文件内容，并且容器的文件系统是可写的，可以在运行时进行修改
> 3. **分层的共享与复用**：由于每个分层只包含自己的修改部分，因此多个镜像可以共享相同的基础分层。这样，**在构建镜像时，可以复用已有的基础分层，只需在其上添加需要的修改层。这种分层的共享和复用机制大大减少了磁盘占用和构建时间**

**特点：**

Docker镜像都是只读的，当一个容器启动时，一个新的可写层被加载到镜像的顶部，这一层就是我们通常说的容器层，容器之下的都叫镜像层。

<img src="C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230711100545108.png" alt="image-20230711100545108" style="zoom:150%;" />

###### 如何本地修改并保存一个自己的镜像——commit镜像

```bash
docker commit -m="提交描述信息" -a="作者" 容器id 目标镜像名[TAG]

#举例
docker run -it --name tomcat01 -p 3030:8080 tomcat
docker exec -it tomcat01 /bin/bash
cp webapps.dist/* webapps
exit
docker commit -a="bluebriage" -m="add webapps" b65099314ce5 tomcat02

以上成功在本地修改了一个tomcat镜像并提交成功

```

#### 容器数据卷

##### 	什么是容器数据卷

> **Docker理念回顾：**
>
> 将应用程序和环境打包成一个镜像，并成功部署
>
> **思考？**
>
> 如果数据都存储在容器当中，那么删除容器数据岂不是会全部丢失！
>
> 所以我们产生了一个持久性的保存数据的需求——**容器间数据共享技术**（容器数据卷）
>
> 容器数据卷是一种特殊的目录，可以在容器和容器主机之间共享和持久化数据。数据卷允许容器内的数据在容器重启、迁移或删除后仍然存在。
>
> 通过将主机上的目录或文件挂载到容器中，可以创建一个数据卷。当容器启动时，挂载的目录或文件会出现在容器的文件系统中，容器可以在该目录下读取和写入数据。
>
> 数据卷的优势包括：
>
> 1. 数据持久性：容器可以在启动、停止和迁移后保留数据，即使容器被删除，数据仍然可以保留在数据卷中。
> 2. 数据共享：多个容器可以共享同一个数据卷，使它们可以访问和共享相同的数据。
> 3. 数据备份和恢复：容器的数据可以轻松备份到主机上的数据卷中，再通过恢复数据卷来还原容器的状态。
> 4. 数据卷驱动：Docker提供了多种数据卷驱动程序，可以根据不同的需求选择适合的驱动程序。

**总结：容器数据卷技术是一种同步技术，是一个特殊的目录，将容器中的目录挂载到指定的地方，可以将数据同步到你指定的目录下，这样就不会丢失**

###### 	使用数据卷

​	方式一：使用命令挂载 -v

> ```bash
> docker run -it -v 主机目录:容器目录
> 
> #举例
> docker run -it -v /home/ceshi:/home centos
> 运行一个centos容器并将主机的/home/ceshi挂载到容器内的/home目录上 两者会持续同步
> 
> #好处
> 可以用作数据备份，也可以远程修改容器内的数据进行某些操作
> ```

##### 实战：安装MySQL

```bash
#拉取镜像
[root@docker /]# docker pull mysql

#数据挂载以及设置MYSQL_ROOT_PASSWORD=
[root@docker /]# docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=000000 mysql

挂载了 /home/mysql/conf:/etc/mysql/conf.d 和 /home/data:/var/lib/mysql



```

###### 具名挂载和匿名挂载

```bash
#匿名挂载

-v 容器内路径
docker run -d -p --name nginx01 -v /etc/nginx nginx

-P 默认情况下，Docker会自动为容器分配一个可用的主机端口，并将其映射到容器的相同端口。这样可以方便地使容器的服务通过随机端口访问。

#这种就是匿名挂载，在-v的时候只指定了容器内的路径，没有写容器外的路径

#具名挂载
docker run -d -P --name nginx03 -v juming-nginx:/etc/nginx nginx


#这种就是具名挂载，在-v的时候指定了卷名:器内路径
docker volume ls

docker volume ls
DRIVER    VOLUME NAME
local     juming-nginx

#查看卷信息
[root@docker ~]# docker volume inspect juming-nginx
[
    {
        "CreatedAt": "2023-07-11T08:06:02-04:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]

发现卷名在/var/lib/docker/volumes/juming-nginx/_data目录下

#注，所有docker容器内的卷，没有指定目录的情况下都是在/var/lib/docker/volumes/xxxx/_data

通过具名挂载可以方便的找到我们的一个卷，大多数情况在使用的具名挂载

#如何分辨是具名挂载、匿名挂载、还是指定路径挂载？
-v 容器内路径   #匿名挂载
-v 卷名:容器内路径 #具名挂载
-v 宿主机路径:容器内路径 #指定路径挂载

扩展：

#通过 -v 

```

> ###### 初识Dockerfile
>
> Dockerfile就是用来构建docker镜像的构建文件，命令脚本
>
> 通过这个脚本可以生成镜像，一个一个的命令构建出一层一层的镜像，每个命令都是一层
>
> ![image-20230712145840988](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230712145840988.png)

这是构成centos镜像的官方代码 官方既然可以制作镜像，那么我们自己也能制作！

#### DokcerFile 制作

#### （Dockerfile文件不要放在根目录下，最好新建一个项目目录） 

基础知识：

1.每个保留关键字（指令）都必须是大写字母

2.从上到下进行执行

3.# 表示注释

4.每一个指令都会创建一层镜像。

![image-20230712150519821](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230712150519821.png)

dockerfile是面向开发的，以后要发布项目，做镜像，就需要编写dockerfile文件。Docker镜像逐渐成为企业交付的标准，必须要掌握。

要了解 Dcokerfile——Dockerimages——Docker容器的关系

> DockerFile：构建文件，定义了一切的步骤，相当于源代码
>
> Dockerimages：最终发布和运行的产品
>
> Dcoker容器：容器就是镜像运行起来的服务器
>
> 开发、部署、运维，缺一不可

###### DockerFile的指令

```bash
FROM  	  				# 指定基础镜像，最下层
MAINTAINER				# 指定作者信息（name+Email）
RUN 				    # 指定需要运行的命令
ADD 				    # 指定要添加的文件的路径
WORKDIR				    # 镜像的工作目录
VOLUME				    # 设置卷，挂载的目录
EXPOSE                   # 暴露端口 相当于 -p
CMD					    # 指定容器启动的时候要运行的命令（只有最后一个CMD会被执行，可被替代）
ENTRYPOINT				# 指定这个容器启动的时候要运行的命令（可以追加命令）
ONBUILD 				# 当构建一个被继承的 DockerFile 这个时候就会运行ONBUILD 的指令 属于触发指令
COPY					# 类似ADD，将文件拷贝到镜像中
ENV 					# 构建的时候设置环境变量 相当于 -e


#构建镜像

docker build -f <dockerfile的绝对路径> -t <镜像名：TAG> <当前路径> （一般用 .）
```

##### 实战：构建centos

Docker Hub 中百分之九十的镜像都是在FROM scratch 基础上

 

```bash
#开始构建

vim mydockerfile    # 进入一个全新文件

FROM centos:7	    # 底层是centos:7镜像

MAINTAINER ChangBoWen<bowenchang@outlook.com> # 作者信息

ENV MYPATH /usr/local  	# 设置环境变量的默认路径

WORKDIR $MYPATH         # 设置镜像的工作目录为MYPATH $是占位符

RUN yum -y install vim  # 设置自动运行 yum ... 命令
RUN yum -y install net-tools

EXPOSE 80			   # 暴露容器内端口为80	

CMD echo $MYPATH	    # 输出MYPATH
CMD echo "-------end------"   # 输出 ------end-----
CMD /bin/bash			# 设置默认为/bin/bash命令行


```

###### 报错：无法直接联网下载vim、net-tools 

猜测：容器/镜像网络问题导致无法解析DNS主机

###### 	CMD 与 ENTRYPOIT 的区别

```bash
#编辑如下dockerfile文件   使用CMD

vim dockerfile

FROM centos:7
CMD ["ls","-a"]

#构建
docker build -f /home/dockeriflr/dockerfile -t ceshi:0.1 .

#运行
[root@docker dockerfile]# docker run -it 06f2cc65ea4a 
.   .dockerenv	       bin  etc   lib	 media	opt   root  sbin  sys  usr
..  anaconda-post.log  dev  home  lib64  mnt	proc  run   srv   tmp  var

#运行时追加命令
[root@docker dockerfile]# docker run -it 06f2cc65ea4a  -l
docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "-l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container:  

报错： “exec: “-l”: executable file not found in $PATH” 指示在 $PATH 环境变量中找不到可执行文件 “-l” 
也就是说 我们写dockerfile文件时CMD了["ls","-a"],运行时如果直接在后面加命令的话是无法识别的

#使用ENTRYPOINT

vim ceshi2

FROM centos:7
ENTRYPOINT ["ls","-a"]

docker build -f /home/dockerfile/ceshi2 -t ceshi2:0.2 .

[root@docker dockerfile]# docker run -it  5184c7d459a0 -l
total 12
drwxr-xr-x.   1 root root     6 Jul 12 09:30 .
drwxr-xr-x.   1 root root     6 Jul 12 09:30 ..
-rwxr-xr-x.   1 root root     0 Jul 12 09:30 .dockerenv
-rw-r--r--.   1 root root 12114 Nov 13  2020 anaconda-post.log
lrwxrwxrwx.   1 root root     7 Nov 13  2020 bin -> usr/bin
drwxr-xr-x.   5 root root   360 Jul 12 09:30 dev
drwxr-xr-x.   1 root root    66 Jul 12 09:30 etc
drwxr-xr-x.   2 root root     6 Apr 11  2018 home
lrwxrwxrwx.   1 root root     7 Nov 13  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root     9 Nov 13  2020 lib64 -> usr/lib64
drwxr-xr-x.   2 root root     6 Apr 11  2018 media
drwxr-xr-x.   2 root root     6 Apr 11  2018 mnt
drwxr-xr-x.   2 root root     6 Apr 11  2018 opt
dr-xr-xr-x. 144 root root     0 Jul 12 09:30 proc
dr-xr-x---.   2 root root   114 Nov 13  2020 root
drwxr-xr-x.  11 root root   148 Nov 13  2020 run
lrwxrwxrwx.   1 root root     8 Nov 13  2020 sbin -> usr/sbin
drwxr-xr-x.   2 root root     6 Apr 11  2018 srv
dr-xr-xr-x.  13 root root     0 Jul 10 00:52 sys
drwxrwxrwt.   7 root root   132 Nov 13  2020 tmp
drwxr-xr-x.  13 root root   155 Nov 13  2020 usr
drwxr-xr-x.  18 root root   238 Nov 13  2020 var

发现 使用ENTRYPOINT 时可以在运行时追加命令

```

##### 发布自己的镜像

```bash
#三步原则

#登录到仓库
docker login -u    -p

#镜像拉取到本地
docker pull 镜像名
或
docker load -i 镜像名

#镜像改名
#镜像上传时，更改后的镜像名必须与自己的hub库保持一致
docker tag 原镜像名 镜像名（ip/分类/镜像名:tag）

#镜像上传
dokcer push 服务器IP：端口/分类/镜像名:tag

```

##### 实战：部署nginx输出网页

```bash
#编写输出内容
vim index.html
xxxxxxxxx 

#配置yum源
vim yum.repo

[yum]
name=yum
baseurl=file:///opt/yum
gpgcheck=0
enable=1

#编写dockerfile
FROM centos:centos7.9.2009
RUN rm -rf /etc/yum.repos.d/*
RUN yum/ /opt/yum  #将本地的yum文件复制到镜像里的yum源里
RUN yum.repo /etc/yum.repos.d/yum.repo #将编写好的yum源复制到镜像里的yum.repo目录下
ADD index.html /usr/share/naginx/html #将编写好的html文件复制到镜像内的html文件里
RUN yum -y install nginx
EXPOSE 80 #内部暴露80端口
CMD ["nginx","-g","daemon off;"] #以非守护进程方式启动nginx服务

#构建镜像
docker build -t nginx666:v1 -f dockerfile .

#运行容器
docker run -d --name nginx -p 8888:80 nginx666:v1

#浏览器地址栏输入 IP+端口号验证

######################另一种实现####################

#创建dockerfile文件
touch /home/dockerfile

#编辑dockerfile
vim /home/dockerfile

FROM nginx #底层直接用nginx

RUN echo '<h1>....<h2>' > /usr/share/nginx/html/nginx.html  #将文本直接写入到nginx的html文件中

EXPOSE 80 #暴露80端口


```

#### 补充

###### 打包镜像为tar（类似于离线下载）

```bash
 docker save [OPTIONS] IMAGE [IMAGE...]


一些常用的选项包括：

-o, --output <文件路径>：指定要保存的 tar 文件的输出路径和文件名。
--quiet, -q：静默模式，不输出额外信息。

以下是一个示例，将名为 myimage 的镜像保存到 myimage.tar 文件中：

docker save -o myimage.tar myimage

完成后，将在当前目录下生成一个名为 myimage.tar 的文件，其中包含了 myimage 镜像的所有层和元数据。
```

###### 将tar镜像文件拉取到本地

```bash
docker load [OPTIONS] 镜像名.tar

一些常用的选项包括：

-i, --input <文件路径>：指定要加载的镜像文件的路径。

以下是一个示例，假设当前目录下有一个名为 myimage.tar 的镜像文件，使用 docker load 命令将其加载到 Docker 中：

docker load -i myimage.tar
这将会读取 myimage.tar 文件并加载其中的镜像和元数据到 Docker 中。
```

#### Docker 网络详解

###### 	理解Docker0

![image-20230714152142284](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230714152142284.png)

**思考：**

​	运行一个容器，Linux主机/服务器，能不能ping通容器内部









#### Docker-compose 

**安装:**

```bash
#运行以下命令下载Docker Compose的当前稳定版本

sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#将可执行权限应用于二进制文件：

sudo chmod +x /usr/local/bin/docker-compose

#创建软链：(复制到bin目录下 在所有情况下都可以使用docker-c)

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

#测试是否安装成功：

docker-compose --version
```





#### Docker machine

**安装：**

```bash
 curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine 
    
chmod +x /tmp/docker-machine &&
    
sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```

