[TOC]

​		古之成大事者，不惟有超世之才，亦必有千韧不拔之志也

# **Kubernetes**

## 什么是云原生？

**原生**

就是任意编程语言开发的应用，即原生应用

**云**

即云环境，提供云服务的基础设施，包括公有云、私有云或混合云

**云原生**

将**原生应用上云**的整个过程，是一种以云计算环境为基础，采用容器化、自动化和微服务架构的软件开发和部署方法

## 什么是云计算？

**云平台**

与云平台对应的就是 "本地"

**为什么选择云平台？**

* 环境统一
* 按需付费
* 即开即用
* 稳定性好
* ........

国内常见的云平台：

阿里云、网易云、华为云、青龙云

国外有：

亚马逊 AWS、微软 AZURE

**计算**

云平台提供的算力资源，以此来执行各种计算任务，包括数据处理、算法运算、模型训练、应用程序执行等

**云计算**

是一种模式，是一种使用云平台的算力资源来进行各种任务的方法，并根据实际需求灵活管理计算资源，提升计算效率和可靠性。

## 历史选择下的K8s

> 随着容器化技术的广泛运用，管理和协调大规模容器集群的复杂性成为了一个问题。
>
> **我们急需一个大规模容器编排系统**

### K8s是什么

​			**Kubernetes**，又称为 k8s（K和S之间有8个字母，所以简称 k8s）或者简称为 "kube" ，是一种可自动实施 [Linux 容器](https://link.zhihu.com/?target=https%3A//www.redhat.com/zh/topics/containers/whats-a-linux-container)操作的开源平台。它可以帮助用户省去应用容器化过程的许多手动部署和扩展操作。也就是说，您可以将运行 Linux 容器的多组主机聚集在一起，由 Kubernetes 帮助您轻松高效地管理这些集群。而且，这些集群可跨[公共云](https://link.zhihu.com/?target=https%3A//www.redhat.com/zh/topics/cloud-computing/what-is-public-cloud)、[私有云](https://link.zhihu.com/?target=https%3A//www.redhat.com/zh/topics/cloud-computing/what-is-private-cloud)或[混合云](https://link.zhihu.com/?target=https%3A//www.redhat.com/zh/topics/cloud-computing/what-is-hybrid-cloud)部署主机。因此，对于要求快速扩展的[云原生应用](https://link.zhihu.com/?target=https%3A//www.redhat.com/zh/topics/cloud-native-apps)而言（例如[借助 Apache Kafka 进行的实时数据流处理](https://link.zhihu.com/?target=https%3A//www.redhat.com/zh/topics/integration/what-is-apache-kafka)），Kubernetes 是理想的托管平台。

##### 文档查阅

> **https://www.kubernetes.org.cn/k8s** k8s中文社区
>
> **https://kubernetes.io/zh-cn/docs/concepts/overview/** k8s官方网站
>
> **https://www.redhat.com/zh/solutions/cloud-native-development** RedHat官方网站
>
> 

##### K8s具有以下特征：

* **服务发现和负载均衡**

  负载均衡：Kubernetes 可以使用 DNS 名称或自己的IP地址公开容器，如果进入容器的流量很大，Kubernetes可以负责均衡并分配网络流量，从而使部署稳定。

  服务发现：提供了一种方便的方式，使得应用程序能够相互发现和建立通信连接，无需关心具体的网络地址。这种方式使得应用程序能够以一种灵活而可靠的方式进行通信，同时，当应用程序的网络地址发生变化时，也能够自动更新连接信息，确保通信的顺利进行。

* **存储编排**

  Kubernetes允许你自动挂载你选择的存储系统，如例如本地存储，公共云提供商等等。存储编排提供了一种统一且可扩展的方式来管理和调度存储资源，使得应用程序的持久化存储变得更加简单和可靠。

* **自动部署和回滚**

  你可以使用Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态更改为期望状态。例如，你可以自动化Kubernetes来为你的部署创建新容器，删除现有容器并将它们的所有资源用于新容器。

* **自动完成装箱计算**

  Kubernetes 允许你指定每个容器所需CPU 和RAM。当容器指定了资源请求时，Kubernetes 可以做更好的决策来管理容器的资源。

* **自我修复**

  Kubernetes重新启动失败的容器、替换容器、杀死不响应用       户定义的运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。

* **密钥与配置管理**

  Kubernetes 允许你存储和管理敏感信息，例如密OAuth令牌和ssh密钥。你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

*Kubernetes 为你提供了一个可弹性允许分布式系统的框架。Kubernetes会满足你的扩展要求、故障转移、部署模式等。例如，Kubernetes 可以轻松管理系统的Canary（金丝雀即灰度发布）部署。*

### 名词解释

##### 集群（Cluster）

> 由多台独立的计算机或服务器（称为节点）组成的一个联网的集合体。这些节点通过网络进行通信和协作，以实现共同的目标或提供共享的资源。
>
> 容器集群：使用容器编排工具（如Kubernetes）将多个节点组成一个集群，以管理和部署容器化应用程序。

##### 节点（Node）

> 指集群中的一个作节点，它可以是一台物理服务器、虚拟机或云实例。**每个节点上可以运行一个或多个Pod**。

##### Pod

> 是Kubernetes中的**最小调度和部署单位，它是一组逻辑上紧密关联的容器集合。**这些容器共享相同的资源（如网络命名空间和存储卷），它们可以协同工作并一起运行应用程序。通常情况下，一个Pod中只运行一个主要容器，其他辅助容器可以与之共享网络和存储资源。

##### control plane（控制面板）

>  是Kubernetes集群的大脑，负责管理和协调整个集群中的各个组件和任务。

##### tuning （调谐）

> 在Kubernetes中，调谐（tuning）指的是**通过调整集群的状态和配置来使其达到更理想的性能、可靠性和可扩展性。**调谐集群状态的目标是使集群在吞吐量、响应时间、资源利用率、容错能力等方面更加高效和稳定。

##### EOF（end of file） 

> 在Linux中，EOF（End of File）是一个特殊的标记，用于**表示输入流的结束**。通常情况下，当你在终端或文本编辑器中按下特定的组合键时，可以输入EOF。
>
> 在终端中，可以使用以下组合键来输入EOF：
>
> - Ctrl + D：在新的一行中按下Ctrl和D组合键即可输入EOF。
> - Ctrl + C：在终端中使用Ctrl和C组合键会终止当前正在运行的程序，但在某些情况下，也可以作为EOF的替代。

##### DaemonSet （守护进程集）

> DaemonSet 是一种资源控制器，用于在集群中的每个节点上运行一个副本。它确保在集群中的每个节点上都运行指定的 Pod 实例，以便满足某些需求或执行特定任务。
>
> 当你在集群中添加或删除节点时，DaemonSet 会自动调整，以确保每个节点上都有一个 Pod 实例在运行。这对于在每个节点上部署日志收集器、监控代理或网络插件等常驻的系统级任务非常有用。

##### tcp socket (tcp 套接字)

> 是应用与TCP网络协议的中间软件抽象层，它是一组接口,是建立在传输控制协议（TCP）之上的一种通信实现，通过使用IP地址和端口号来标识和定位不同的应用程序或进程。

### K8s架构

##### 1，工作方式

**K8s Cluster = N 个 Master 节点 + N 个 Worker 节点       N>=1**

Kubernetes的工作方式如下：

1. 用户通过Kubernetes API与**Master节点**进行交互，创建、修改或删除应用程序的描述文件（如Deployment等）。
2. Master节点接收到请求后，通过**Scheduler**将**Pod**调度到合适的节点上运行，并将相应的任务分配给Kubelet进程。
3. **Kubelet**负责在节点上创建和管理Pod，并通过**Container Runtime**运行容器。
4. **Service**实现了Pod的发现和负载均衡，将请求路由到正确的Pod上。
5. **Master**节点不断监控和更新集群状态，确保应用程序的可用性和健壮性。
6. 当应用程序需要扩展时，用户可以通过扩展Pod或增加节点的方式来进行水平扩展，Kubernetes会自动进行负载均衡和容器的重新分配。

总的来说，Kubernetes的工作方式就是通过Master节点对整个集群进行管理和控制，将工作负载分布到节点上，并提供服务发现和负载均衡等功能，以实现高度可靠和可扩展的容器化应用程序管理。

##### 2，组件架构

理解Kubernetes的组件架构是理解整个系统如何协同工作的关键。Kubernetes的组件架构可以分为控制平面（Control Plane）和节点（Nodes）两部分。

![img](https://pic3.zhimg.com/80/v2-ef7d2cbd9a4722cc4efd115342542ade_1440w.webp)

![image-20230906195651185](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230906195651185.png)

1. 控制面板（Control Plane）：
   - API Server（API 服务器）：作为控制面板 的入口，接收来自用户和其他组件的API请求，并处理和验证这些请求。
   - Scheduler（调度器）：负责根据容器的资源需求和其他约束条件将Pod分配到合适的节点上运行。
   - Controller Manager（控制器管理器）：管理各种控制器，监视集群中对象的状态，保持集群状态与期望状态的一致性。
   - etcd（分布式键值存储）：作为数据存储，保存集群的配置信息、状态信息以及元数据。

2. 节点（Nodes）：
   - Kubelet（kubelet）：运行在每个节点上，负责管理节点上的容器，与Master节点通信，并确保Pod的规约状态与期望状态一致。
   - Container Runtime（容器运行时）：负责运行容器，如Docker、containerd等。
   - kube-proxy（代理）：负责管理节点上的网络代理和负载均衡，以提供服务发现和负载均衡功能。

3. 其他核心组件：
   - CoreDNS（域名系统）：为集群提供DNS服务，支持服务发现和服务解析。
   - Ingress Controller（入口控制器）：负责管理到集群内部服务的流量路由。
   - Dashboard（仪表板）：提供用户图形化的Web界面，用于操作和监控集群。
   - Metrics Server（指标服务器）：收集并存储集群中各个组件的性能指标。
   - Logging and Monitoring（日志和监控）：用于集群的日志收集和监控系统。

这些组件协同工作，形成一个高度可扩展、弹性和自动化的容器管理平台。控制平面负责管理和控制整个集群的状态和行为，而节点上的组件负责运行容器和提供服务。通过定义的Kubernetes对象，用户可以在集群中部署应用程序，并通过API接口进行管理和监控。

##### 3，组件详解

**Kubernetes集群主要由Master和Node两类节点组成**

​	**Master节点组件：**

> - **kube-api-server**：（）
>   - 作为集群的API服务器，处理请求并将其转发给相应的组件进行执行。
>   - 管理集群状态和配置的持久化存储。
> - **kube-controller-manager**：（维护集群状态）
>   - 包含一组控制器，用于管理集群的运行状态。
>   - 不同的控制器负责监视和调谐集群的不同方面，如节点、副本、服务等。
>   - 确保集群的配置和状态与期望的一致。
> - **kube-scheduler**：（资源调度）
>   - 将新创建的Pod调度到适当的节点上的组件。
>   - 考虑节点资源的可用性、Pod的要求和调度策略来选择最佳节点。
>   - 目标是充分利用资源、负载均衡和提供高可用性。
> - **etcd**：（保存集群状态）
>   - Kubernetes控制平面的后备存储系统。
>   - 存储集群的配置数据和状态信息。
>   - 提供分布式的键值存储，确保数据的一致性和可靠性。
>   - 各组件通过与etcd通信共享和同步集群状态。

​	**Work节点组件：**

> * **Kubelet**：Kubelet是运行在每个工作节点上的代理组件，负责与Master节点通信，并管理该节点上运行的Pod。它会监测分配给它的Pod规格（定义在API服务器上），确保Pod的容器正常运行并符合集群的状态。
>
> * **Container Runtime**：Container Runtime（容器运行时）是在工作节点上运行容器的软件，它负责管理和执行容器。Kubernetes支持多种容器运行时，如Docker、containerd和CRI-O等。容器运行时将容器镜像转换为运行中的容器，并提供管理容器生命周期、网络和存储等功能。
> * **Kube-proxy**：Kube-proxy负责为Pod提供网络代理和负载均衡。它运行在每个工作节点上，监视集群服务的创建和删除，并配置相应的网络规则，以便在集群内部和外部提供透明的服务访问。Kube-proxy还负责处理网络包装和目标地址的转发。
> * **DNS**：在每个工作节点上，Kubernetes集群通常配置了一个DNS组件（如CoreDNS） 吧，它负责提供集群内部的域名解析服务。每个Pod都被分配了唯一的域名，其他Pod可以通过该域名来解析和访问。

**设计架构图**

![image-20230912191529757](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230912191529757.png)

### K8s集群搭建（非二进制）

##### 1、Docker 容器化环境安装

```bash
yum -y install yum-utils

#阿里云Docker源
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#安装docker
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#启动docker
systemctl start docker

#配置Docker镜像加速
[root@docker ~] mkdir -p /etc/docker
登录阿里云 拉取自己账号的加速地址 如下
[root@docker ~]sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://088klgwj.mirror.aliyuncs.com"]
}
EOF
[root@docker ~] sudo systemctl daemon-reload  #重新加载某个服务的配置文件
[root@docker ~] sudo systemctl restart docker

#启动docker
[root@docker ~] systemctl start docker
```

##### 2、设置预备环境

​	

```bash
#更改节点主机名
hostnamectl set-hostname k8s-master-node1
hostnamectl set-hostname k8s-work-node1

#关闭交换分区（swap为0）
##swap相当于windows中的虚拟内存，开启swap会严重损耗性能，一般情况下推荐gu
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab

#禁用selinux     #selinux是Linux的一个自带的安全系统
setenforce 0

或者

vi /etc/selinux/config
selinux=permissive     (宽容模式)

#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf  
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables =1
net.bridge.bridge-nf-call-iptables =1
EOF

##补充：
##tee命令用于显示程序的输出并将其写入指定文件中，通常搭配管道符使用 如:ls -l | tee 123.txt 是将ls -l 输出的东西写入到123.txt文件中。
##cat <<EOF......EOF 是shell脚本中常用的写法，将以下多行文本作为输入传递给后续的命令，并在此示例中将其写入/etc/modules-load.d/k8s.conf文件中。


sudo sysctl --system

```

##### 3、安装集群三大件

```bash
#配置yum源
cat <<EOF | sudo tee /etc/yum.repo.a/kubernetes.repo
[kubernetes]
name=kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-e17-x86_64
enable=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

#安装三大件
##kubeadm
sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.0 --disableexcludes=kubernetes

#启动kubelet
sudo systemctl enable --now kubelet 

值得注意的是：Kubelet现在每隔几秒就会重启，因为它陷入了一个等待kubeadm指令的死循环
```

##### 4、用kubeadm引导集群

###### 	1，下载各个机器所需的镜像

```bash
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-aposerver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF

chmod +x ./images.sh && ./images.sh

#检查一下有七个镜像
docker iamges 
```

###### 	2，初始化主节点

```bash
#所有机器添加master域名映射，以下需要修改为自己的内网ip
echo "xxx.xxx.xxx.xxx cluster-endpoint" >> /etc/hosts
echo "xxx.xxx.xxx.xxx master" >> /etc/hosts
echo "xxx.xxx.xxx.xxx node01" >> /etc/hosts

#主节点运行
kubeadm init \
--aposerver-advertise-address=10.140.122.4 \
--control-plane-endpoint=cluster-endpoint \
--images-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/
```

### 资源管理

##### 	YAML 文件

​		YAMl（http://yaml.org/）是专门用来写配置文件的语言，非常简洁强大，远比JSON格式方便。YAML语言的设计目标，就是方便人类读写。它实质上是一种通用的数据串行化格式

​		**基本语法：**

* 大小写敏感
* 使用缩进表示层级关系
* 缩进时**不能使用TAB键**，只能使用空格
* 缩进的格数不重要，只要相同层级的元素左对齐即可
* “#” 表示注释

在kubenetes中，需要了解两种YAML结构类型

* Lists
* Maps

基本数据类型

**纯量：**

最基本的，不可再分的值，包括:字符串	，布尔值，整数，浮点数，null，时间，日期

​	实例：

```yaml
## 注释
name: xx (注意空格)
image: ~ (表示空值null)

## yaml中如何表示日期和时间
Date: 2022-09-11
Time: 2022-09-11T18:36+08:00     (日期后面跟时间，要写一个大写的T然后加时区（东八区：08：00）)

## yaml中如何处理长字符串
方法一：
singLingString: >
  this is a very long String
  another xxxx
  another xxxx
# 要输入长字符串时要先写一个 > 然后接下来的每行要左对齐
#机器识别到的就是 前面每行后保留一个空格，最后一行后保留一个换行符"this is a very long String another xxxx another xxxx\n"
方法二：
multiLineString: 
  this is a very long String
  another xxxx
  another xxxx
#机器识别为"this is a very long String\nanother xxxx\n another xxxx\n" 保留了每行的换行符
```

**列表（list）：**

​	以-开头的行表示构成一个列表                                       

```yaml
ports:  
 - 6479 #列表一
   7778
   2222
 
 - 6380 #列表二
   1212
   2121
   2332
 
```

**对象：**

```yaml
#一个总的父亲，下面是缩进统一的儿子，这就构成了一个对象
container: 
 name: mysql
 image: mysql
 prot: 8080
 version: v1
```



### k8s的三种资源管理方式

* 命令式对象管理：直接用**命令**去操作k8s资源 

```shell
##以管理一个pod资源为例
kubectl run nginx-pod --image=nginx:1.17.1 --port=80

#运行了一个启动一个名为nginx-pod的pod的命令，并指定了pod内容器的镜像为nginx:1.17.1，向外暴露80端口
```

* 命令式对象配置：通过命令配置和**配置文件**去操作k8s资源（不太灵活）

```shell
##创建一个配置文件为nginx-pod.yaml的pod
kubectl create -f nginx-pod.yaml
#这个命令会创建一个全新的以此yaml文件为蓝本的pod，如果此资源已经存在或者被定义过，则会报错。
```

* 声明式对象配置：通过**apply命令**和配置文件去操作k8s资源（建议使用）

```shell
kubectl apply -f nginx-pod.yaml

#如果以此yaml为蓝本的资源此前已经存在，则会更新资源；如果不存在，则创建新的资源，相对来说更加灵活值得推荐。

# -f 的作用是表示后面是一个yaml文件，让kubectl去读取它并创建，更新，删除d的资源。
```

#### 命令式对象管理

**kubectl命令**

​	kubectl是kubenetes集群的命令行工具，通过它能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署，kubectl的语法如下：

```shell
kubectl [command] [type] [name] [flags]
```

**command:**指定对资源执行的命令，如create、get、delete

**type：**指定资源类型，比如deploymen、pod、service

**name：**指定资源的名称，名称大小写敏感

**flags：**指定可选的参数

例如：

```shell
# 查看所有的pod
kubectl get pod

#查看某个pod
kubectl get pod pod_name

#查看某个pod，以yaml格式展示结果
kubectl get pod pod_name -o yaml

```

kubectl常见的可执行命令（command）：

![image-20230922090917759](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230922090917759.png)



![image-20230922091047649](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230922091047649.png)



kubectl常见的资源（type）：

![image-20230922091525131](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230922091525131.png)



![image-20230922091730794](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230922091730794.png)



​	**命令式对象管理的案例演示：**

​		下面以一个namespace的创建和删除简单演示下命令的使用

```shell
namespace （命名空间），就像一个一个的独立的空间，彼此之间的操作是隔离的，互不影响，互不相通的

#创建一个namespace，名为dev
kubectl create namespace dev

#在dev下创建并运行一个nginx的pod
kubectl run pod pod_9_22 --image=nginx -n dev

#查询创建的pod
kubectl get pod -n dev

#删除pod
kubectl delete pod pod_9_22 -n dev

#删除namespace
kubectl delete namespace dev

严格遵循 kubectl [command] [type] [name] [flags]

#namespace 可简写为ns
```



#### 命令式对象配置

​	命令式对象配置就是使用**命令配合配置文件**一起来操作kubenetes资源

​	创建一个nginxpod.yaml 内容如下:

```yaml
vim nginxpod.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod2
  namespace: dev

spec:
  containers:
  - name: nginx-containers2
    image: nginx
```

命令式对象配置，可以使用多个命令进行资源管理

**执行create命令，创建资源**

```shell
kubectl create -f nginxpod.yaml
```

此时发现，此yaml文件创建了两个资源对象，分别是namespace和pod

**执行get命令，获取资源**

```shell
kubectl get -f nginx.yaml

#可查看运行状态
```

**执行delate命令，删除资源**

```shell
kubectl delate -f nginx.yaml

#此时发现两个资源都被删除了


```

```markdown
### 总结：
		命令式对象配置的方式操作资源，可以简单的认为：命令 + yaml配置文件

```



#### 声明式对象配置

​	与命令式对象配置很相似，区别在于 但它只有一个命令 apply。

​	继续以nginxpod.yaml为例：

```shell
#首先执行一次 kubectl apply -f nginxpod.yaml 发现创建了对应资源
kubectl apply -f nginxpod.yaml

namespace/dev created
pod/nginxpod created

#再执行一次，提示为 unchanged 没有改变
kubectl apply -f nginxpod.yaml 

namespace/dev unchanged
pod/nginxpod unchanged

#改动yaml文件，提示被修改
kubectl apply -f nginxpod.yaml 
namespace/dev unchanged

pod/nginxpod configured


```



```markdown
#### 总结：
		 声明式对象配置就是使用apply描述一个资源最终的状态
		 使用apply操作资源：
		 	如果资源不存在，就创建，相当于 kubectl create
		 	如果资源存在，就更新资源，相当于 kubectl patch
```



> 使用推荐：三种资源管理方式应该怎么用？

创建/更新资源   使用 声明式对象配置 kubectl apply -f xxx.yaml

删除资源   		 使用 命令式对象配置 kubectl deleate -f xxx.yaml

查询资源     	   使用 命令式对象管理 kubectl get/describe 资源名称

> 扩展：kubectl 如何在worker节点上运行

kubectl 的运行需要配置文件，它的配置文件是 ~/.kube,需要将master上的.kube文件复制到worker节点上，即在master节点上执行下面操作：

```shell
scp -r ~/.kube  k8s-worker-node1:~/
```



#### Namespace 命名空间

  在k8s中，namespace是一种将集群内部的资源进行隔离的机制，它可以将一个物理的k8s集群划分为多个虚拟的集群，每个ns中的资源都只在该ns中可见，不同的ns之间相互隔离

​	如果不想让两个pod互相访问，那就将两个pod划分到不同的ns中。

​	

![image-20230923113519706](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230923113519706.png)



##### 管理Namespace命名空间

```shell
#查看当前有多少namespace
kubectl get ns

NAME              STATUS   AGE
dashboard-cn      Active   68d
dashboard-en      Active   68d
default           Active   68d #所有pod默认创建在default里
hjsada            Active   25h
kube-node-lease   Active   68d #集群节点之间的心跳维护
kube-public       Active   68d #公开的命名空间，资源可以被所有人访问
kube-system       Active   68d #由k8s系统创建的资源都处于这个命名空间

#下载工具切换命名空间
wget ftp://ftp.rhce.cc/cka-tool/kubens -P /bin/  #下载kubens 到 /bin目录下
chmod +x /bin/kubens


#查看当前所在命名空间
kubens

#创建一个新的命名空间
kubectl create ns ns111

#删除一个命名空间
kubectl delete ns ns111

#切换到ns111命名空间
kubens ns111

#查看namespace详情
kubectl describe ns default

Name:         default
Labels:       kubernetes.io/metadata.name=default
Annotations:  <none>
Status:       Active ## 激活状态，如果是Terminating 则为正在删除状态 

No resource quota.  #resource quota 是针对ns做的资源限制

No LimitRange resource. #limitRange resource 是针对ns中的每个组件做的资源限制

```



##### namespace的配置文件如何书写

```yaml
首先准备一个yaml文件

vim ns-dev.yaml

apiVersion: v1
king: Namespace
metadata:
  name: dev
```



**总结：**

​		Namespace 是k8s中的一种隔离机制，主要用于给不同的租户分配对应资源的命名空间，ns之间是互相隔离的，k8s可以比较高效的分配和管理namespace。





### 深入了解Pod

Pod是kubenetes集群进行管理的最小单元，程序要运行必须部署在容器中，而容器必须存在于Pod中。

Pod可以认为是容器的封装，一个Pod中可以存在一个或多个容器。![Pod](C:\Users\Blue Bridge\Pictures\Pod.png)

kubenetes在集群启动后，集群中的各个组件也都是以Pod方式运行的



**查看，创建，删除Pod**

```shell
查看Pod

#查看当前命名空间有多少pod
kubectl get pods

#查看指定命名空间有多少pod
kubectl get pods -n ns_name

#查看所有命名空间的pod
kubectl get pods -A

#查看pod运行在哪个节点上
kubectl get pods -owide



命令行创建pod

#命令行方式创建Pod
kubecetl run Pod名称 [参数]

参数有：
--image    指定Pod的镜像   
--port=  指定向外暴露的端口
--lables   标签
--dry-run  尝试运行但不运行
--image-pull-policy=  镜像下载策略 默认下为always 即即时本地已经有了这里指定的镜像，但仍然会从网络上下载若指定为 IfNotPresent,则优先使用本地镜像，如果本地没有，才回去下载
--namespace 指定命名空间

#创建一个nginx镜像的pod
#注意默认的nginx镜像只开放80端口 可以在dockerfile中更改
kubectl run test01 --image=nginx --port=80 --image-pull-policy=IfNotPresent

#访问一下这个pod
kubectl get pod nginx01 -owide
test01                     1/1     Running             0               6m50s   10.244.0.56   k8s-master-node1   <none>           <none>

curl 10.244.0.56:80  #访问pod的ip及容器暴露的端口

<!DOCTYPE html>      #nginx输出的内容
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


删除pod
#删除指定pod
kubectl delete pod pod_name --force   #--force可加快删除的速度

```



#### 生成yaml文件创建pod

更建议使用yaml文件来创建pod，这样可以在yaml文件里配置各种属性，生成yaml文件的命令如下：

```yaml
kubectl run pod_name --image=xx  -0 yaml > pod.yaml

#以yaml文件格式输出，然后把结果重定向到pod.yaml文件中，这样就会生成一个模板

apiVersion: v1               #必选，版本号，例如v1
kind: Pod                    #必选，资源类型，例如pod
metadata:					 #必选，元数据
  creationTimestamp: "2023-09-24T08:51:26Z"   #可选，时间戳
  labels:					 #自定义标签列表
    run: test01
  name: test01				 #必选，pod名称，不允许大写
  namespace: default         #pod所属的namespace
  resourceVersion: "514117"  #这是该 YAML 文件所代表的 Kubernetes 资源的版本。它用于 Kubernetes 跟踪资源的更改。
  uid: a10c1c59-30c7-4b92-8cf2-a51adc7665ac #这是 Pod 的唯一标识符。
spec:						 #必选，pod中容器的详细定义
  containers:                #必选，pod中容器列表
  - image: nginx			 #必选，容器的镜像名称
    imagePullPolicy: IfNotPresent  #镜像拉取策略
    name: test01			 #必选，容器名称
    ports:                   #端口暴露列表
    - containerPort: 80		 #容器对外暴露的端口
      protocol: TCP			 #端口协议
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-scrzm
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  
  #创建此Pod
  
  kubectl apply -f pod.yaml
```

​	Pod 配置文件简单示例

```yaml
vim Pod.yaml    ##创建YAML文件
apiVersion: v1  ## V要大写 根据本地集群环境的apiVersion版本
kind: Pod       ## 大写P
metadata:
  name: my-pod  ##除 - . 外不能有特殊字符
  lables:         ##k8s中标签表示为lables
    app: my-app    ##app是标签键，my_app是标签值，采用的是（key:value）键值对的方式，这样可以将Pod分类，这里的类型是app，则此pod可以跟所有app类型的资源关联，进行管理。
spec:
  containers:   ##containers 这里的container加s
  - name: my-container
    image: naginx  ##这里的image不加s
    ports:
    - containerPort: 80  (ports需要再创建一个层级，因为可能有多个端口映射)
```

注：标签的正确拼写 **apiVersion Pod    labels   containers  image  ports   containerPort**

配置文件命名时，只能使用，小写字母、数字、连字符（-）或句点（.）

**使用explain命令查看资源的可配置项**

```markdown
#如果想查看pod下有多少一级参数
kubectl explain pod

FIELDS:
   apiVersion	<string>    版本，每个资源的版本号可以用kubectl explain 资源 来查询到

   kind	<string>            类型，查询方法为 kubectl api-resources

   metadata	<Object>        元数据，资源的名称，所属空间 ，标签分类等
     
   spec	<Object>		    描述，这是配置中最重要的一部分，是对各种资源配置的详细描述
     spec常见的子属性：
     	container <[]object>   容器列表，用于定义容器的详细信息
     	nodeName <string>      根据nodeName的值将pod调度到指定的Node节点上
        nodeSelect <map[]>     根据NodeSelect中定义的信息选择将该pod调度到包含这些label的Node上
        hostNetwork <boolean>  是否使用主机网络模式，默认为false
        volumes <[]object>     存储卷，用于定义pod上面挂载的卷信息
        restartPolicy <string> 重启策略，表示pod遇到故障的时候的处理策略
        
        
   status	<Object>       状态信息，里面的内容由kubenetes自动生成
     
#如果想查看metadata里有多少选项
kubectl explain pods.metadata

#如果想查看spec下的containers有多少选项
 kubectl explain pods.spec.containers
```



#### Pod.spec.containers 文件配置

本节主要研究 *`pod.spec.containers`*属性，这也是pod配置中最为关键的一项配置

```yaml
[root@k8s-master-node1 script]# kubectl explain pod.spec.containers
KIND:     Pod
VERSION:  v1
RESOURCE: containers <[]Object>

FIELDS:
  name    <string>   #容器名称
  image   <string>   #镜像名称
  imagePullPolicy  <string>  #镜像拉取策略
  command <[]string> #容器的启动命令列表，如不指定，使用打包时使用的启动命令
  args    <[]string> #容器的启动命令所需要的参数列表
  env     <[]object> #容器环境变量的配置
  ports   <[]object> #容器需要暴露的端口列表
```

##### **imagePullPolicy**

用于设置镜像拉取的策略，k8s支持三种镜像拉取策略：

* Always：总是从远程仓库拉取镜像
* IfNotPresent：本地有则优先拉本地的，本地无再从远程仓库拉
* Never：只使用本地镜像，本地没有就报错。

> 默认值说明：
>
> ​		如果镜像tag为具体的版本号，默认策略是：IfNotPresent
>
> ​         如果镜像tag为：latest，默认是Always

##### **command**

用于在pod中的容器初始化完毕之后运行一个命令。

例如：

```yaml
spece: 
  containers:
  - name:
    image:
    imagePullPolicy:
    command: ["/bin/bash","touch /tmp/text.txt";"while true;do /bin/bash $(data +%T) >> /tmp/text.txt;sleep 3;done;"
    
    #代码解释，先是指定脚本环境，然后创建一个文件，执行死循环，向文件里重定向当前的时间，然后每输出一个时间戳，停止三秒，无限循环
    
    command字段通常与args字段一起使用，args字段用于指定命令的参数。例如：

spec:
  containers:
    - name: my-container
      image: my-image
      command: ["echo"]
      args: ["Hello,", "World!"]
#  在这个示例中，容器启动后，将执行echo Hello, World!命令。
```

##### **env**

设置pod的容器环境变量列表

```yaml
spece: 
  containers:
  - name:
    image:
    imagePullPolicy:
    command:
    env: #设置环境变量列表
    - name: "username"
      value: "admin"
    - name: "password"
      value: "123456
```

##### **ports 端口设置**

```yaml
kubectl explain pod.spec.containers.ports		##explain命令查看ports详情
KIND:     Pod
VERSION:  v1
RESOURCE: ports <[]Object>
FIELDS:
   containerPort				#容器要暴露的端口（0<x<65536)

   hostIP	<string> 			#w外部端口绑定到主机的ip 一般省略
     
   hostPort	<integer>			#容器在主机上公开的端口，一般省略

   name	<string>				#端口名称，如果指定，必须要保证nmae在pod里是唯一的
  
   protocol	<string>			#端口协议，必须是UDP，TCP或SCTP。默认为TCP

###举例
spece: 
  containers:
  - name:
    image:
    ports: #设置容器暴露的端口列表
    - name: nginx-port
      containerPort: 80
      protocol: TCP
      
### 创建pod 
kubectl run test1 --image=nginx:latest --run-dry=client -o yaml > pod.yaml

###更改容器内的端口暴露为80
spec:
  containers:
  - image: nginx:latest
    imagePullPolicy: IfNotPresent
    name: test1
    ports:
    - containerPort: 80
      protocol: TCP 
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

###运行pod
kubectl apply -f pod.yaml

###查看pod内的端口暴露信息
kubectl describe pod test1

Containers:
  test1:
    Container ID:   docker://c25ed82bd7a72491e36fa58b104ec940aeea63def6664ae35955006935236f9e
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:32da30332506740a2f7c34d5dc70467b7f14ec67d912703568daff790ab3f755
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 07 Oct 2023 11:10:13 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
```

**访问容器中的程序需要使用的是 *podip:containerPort***

##### resources  资源配额

​	容器中的程序要运行，就需要占据一部分资源，比如cpu和内存等，如果不对某个容器的资源做限制，那么它就可能吃掉大量资源，导致其他容器无法运行，针对这种情况，k8s提供了对内存和cpu的资源进行配额的机制，这种机制主要通过resource选项实现，它有两个子选项：

* limits：用于限制运行时容器的最大占用资源，当容器占用资源超过limits时会被终止，并进行重启
* requests：用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动

编写测试案例：

​	

```yaml
apiVersion: v1
kind: pod
metadata:
  name: pod-resource
  namespace: deployns
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresnet
    resource:    #资源配额
      limits:    #容器资源请求上限
        cpu: "2"  #core(核心数)，限制两核
        memory: "10Gi"  #内存限制
      requests:  #容器资源请求下限
        cpu: "1"
        memory: "10Mi"
```

cpu以及memory的单位说明：

* cpu：core，核心数，可以是整数和小数
* memory：内存大小，单位为 Gi、Mi、G、M等 （1Gi = 1024Mi，1G = 1000 M）



#### Pod生命周期（lifecycle）

​	一般将pod对象从创建到终止这段时间范围称为pod的生命周期，它主要包含以下的过程：

* pod创建过程

* 运行初始化容器（init container）过程

* 运行主容器（main container）过程 

  - 容器启动后钩子（post start）、容器终止前钩子（pre stop）
  - 容器的存活性探测（liveness probe）、就绪性探测（readiness probe）

* pod终止过程

  

pod生命周期示意图

![image-20231007195842118](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20231007195842118.png)

在整个生命周期中，pod会出现5种相位 （Pod phase）

下面是 phase 可能的值：

- 挂起（Pending）：Pod 已被 Kubernetes 系统接受，但有一个或者多个容器镜像尚未创建。等待时间包括调度 Pod 的时间和通过网络下载镜像的时间，这可能需要花点时间。
- 运行中（Running）：该 Pod 已经绑定到了一个节点上，Pod 中所有的容器都已被创建。至少有一个容器正在运行，或者正处于启动或重启状态。
- 成功（Succeeded）：Pod 中的所有容器都被成功终止，并且不会再重启。
- 失败（Failed）：Pod 中的所有容器都已终止了，并且至少有一个容器是因为失败终止。也就是说，容器以非0状态退出或者被系统终止。
- 未知（Unknown）：因为某些原因无法取得 Pod 的状态，通常是因为与 Pod 所在主机通信失败。

##### pod生命周期的创建和终止

![image-20231008135224753](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20231008135224753.png)

##### 初始化容器

​	初始化容器是一个名词，指的是在pod的主容器运行之前要运行的容器，主要是做一些前置工作，它具有两大特征：

1.  	初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么kubernetes需要重启它直到成功完成
1.  	初始化容器必须按照定义的顺序执行，当且仅当前一个成功之后，后面的才能运行

初始化容器有很多应用场景，下面列出的是最常见的几个：

* 提供主容器镜像中不具备的工具程序或自自定义代码
* 初始化容器要先于应用容器串行启动并运行完成，因此可用于延后应用容器的启动直至其依赖的条件得到满足

案例1：

​	假设要以主容器来运行nginx，但是要求运行nginx之前先要能够连接上mysql和redis所在服务器

```bash

```



##### pod的延期删除

​	前面讲过，删除pod的时候，可以加上 -- force 选项提高删除pod的速度，那么如果不加--force选项，为什么会变慢呢

​	原因在于kubenetes对于pod的删除有个延期删除期（宽限期）这个时间默认为30s。

​	假如没有宽限期，如果某个pod正在执行用户的某个请求，而我们发出一个删除pod的命令,这个pod会被强制删除，不管它是不是正在处理任务，这叫做粗暴地删除pod

​	那什么是 **优雅地删除pod** ？

​	即当我们对某个pod发出删除命令地时候，这个pod的状态会变成"terminating" (终止) 但此时不会立刻删除这个pod，而是等待它继续处理手头的任务，如果在30s内能够任务完成的话，则pod会被自动删除，如果超过30s还未完成，则会被强制删除，这就叫做优雅地删除pod

​	删除pod地宽限期可以通过参数 **terminatingGracePeriodseconds** 来指定

​	

```yaml
 apiVersion: v1
 kind: Pod
 metadata:
   labels:
     run: test111
   name: test111
   namespace: dev
   creatiionTimestamp: null
 spec:
   terminationGracePeriodSeconds: 15   #宽限期修改为15s
   containers:
     - image: busybox
       imagePullPolicy: IfNotPresent
       name: pod1
       command: ["sh","-c","sleep 1000"] # 执行命令 1000s后命令终止，所以超出了宽限期，理论上delete时15s后pod就会被删除
       resources: {}
```

​	不过，如果容器里运行的是nginx镜像，情况就不一样了，当我们对nginx镜像发出删除信号的时候，pod里的nginx进程会被很快的关闭（fastdown），之后pod也会被很快删除，并不会使用kubenetes的删除宽限期

 但是这也带来了一个问题，如果删除nginx pod的时候有客户正在访问pod怎么办，那客户端就会报错 ，这就引出了下面的 **pod 钩子** 来解决问题

##### pod hook （钩子）

​	在整个pod生命周期（lifecycle）内，有两个hook是可用的

（1）postStart （初始化钩子）：当创建pod的时候，会随着pod里的主程序同时运行，没有先后顺序

 （2）preStop （终止化钩子）： 当删除pod的时候，要先运行preStop里的程序，之后再关闭pod

​	钩子处理器支持使用下面三种方式定义：

* exec命令：在容器里执行一次命令

  ```yaml
  -- 
    lifecycle:
      postStart:
        exec:
          command:
          - cat
          - /tmp/healthy
  --
  ```

* tcpSocket：在当前容器尝试访问指定的socket

  ```yaml
  ##socket 是应用与TCP网络协议的中间软件抽象层，它是一组接口,是建立在传输控制协议（TCP）之上的一种通信实现，通过使用IP地址和端口号来标识和定位不同的应用程序或进程。
  --
    lifecycle:
      postStart:
        tcpSocket:
          port: 8080
  ```

* httpGet：在当前容器中向某url发起http请求

  ```yaml
  --
    lifecycle:
      postStart:
        httpGet:
          path:  #url地址
          port: 80 
          host: 192.168.120.99 #主机地址
          scheme: HTTP #支持的协议，http或https
  ```

**pod hook 案例实现**

```bash
vim podhook.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test101
  name: test101
  namespace: dev
spec:
  containers:
  - image: nginx:latest
    imagePullPolicy: IfNotPresent
    name: c1
    lifecycle:     ##定义容器的生命周期
      postStart:   ##初始化钩子
        exec:      ##定义方式为进入式定义
          command: ["/bin/bash","-c","echo postStart ...  >  /usr/share/nginx/html/index.html"]      ##命令为，进入/bin/bash环境，打印 postStart ... 定向到 nginx的/usr/share/nginx/html/index.htm 覆盖原有的html文件   "-c"是/bin/bash 后面的一个选项 表示后面所跟的是要执行的命令
      preStop:     ##终止化钩子
        exec:
          command: ["/usr/sbin/nginx","-s","quit"]   ##容器关闭前先运行这个钩子，内容是先关闭nginx服务再关闭容器
          
          
kubectl apply -f podhook.yaml

kubectl get pods -n dev -owide

curl pod_ip:80
postStart ...


案例实现成功
```



##### 容器探测

​	容器探测用于检测容器中的应用实例是否正常工作，是保障业务可用性的一种传统机制。如果经过探测，实例的状态不符合预期，那么k8s就会把该问题实例 "摘除"，不承担业务流量，k8s提供了两种探针来实现容器探测：

* liveness Probes ：  存活性探针，用于检测应用实例当前是否处于正常运行状态，如果不是，k8s会重启容器
* readiness Probes：就绪性探针，用于检测应用实例当前是否可以接受请求，如果不能，k8s不会转发流量

> liveness Probes 决定是否重启容器 ，readinessProbe决定是否将请求转发给容器

这两种探针都支持三种探测方式：

* Exec命令：在容器内执行一次命令，如果命令执行的退出码为0，则认为程序正常，否侧不正常

  ```yaml
  -- 
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
        
  --
  ```

  这里只了解一种就行了



#### pod的基本操作

![pod操作](D:\myPicture\Cloud\pod操作.png)

##### **在容器中执行命令**

```bash
kubectl exec pod_name -n ns_name -- 命令

#举例

##1.查看pod里对应目录下的内容
kubectl exec test1 -n dev -- ls /usr/share/nginx/html 
50x.html    #注意 -- 后面只能跟在容器内里执行的linux命令，相应的kubectl命令要写在--前面
index.html

##2.物理机和容器内的内容相互拷贝
kubectl cp /opt/script/ test1:/opt/ -n dev  ##物理机往容器内拷贝
kubectl exec test1 -n dev -- ls /opt/

Kubectl cp test1:/usr/share/nginx/html /opt/ -n dev
##容器内往物理机上拷贝   提示：tar: Removing leading `/' from member names 为正常现象 实际已经拷贝成功

##3.进入pod里并获取bash
kubectl exec  test110 -it -n dev -- bash
root@test110:/# 
```

如果pod里有很多个容器，那默认进入第一个

![kubectl exec](D:\myPicture\Cloud\kubectl exec.png)

```BASH
##4.如果想进入第二个容器的话，用 -c 指定容器名
kubectl	exec test110 -it -n dev -c c2 -- bash

##5.查看pod日志
kubectl logs test110 -n dev

#如果有pod两个容器，用-c 指定
kubectl logs test110 -n dev -c c2

##6.删除pod
kubectl delete pod pod_name 
kubectl delete -f pod.yaml

```

 

#### 手动指定pod运行位置

##### 给节点设置标签

​	当我们创建一个pod时，master会根据自己的算法来决定创建在哪个节点上，所以pod运行在哪个节点上只有创建出来以后才知道。	

​	我们可以在每个节点上设置一些标签，然后指定pod运行在特殊标签的节点上，就可以手动指定pod运行在哪个节点上。

​	标签的格式:: key=value.key的值里可以包括符号"/"或者"."，多个标签用逗号隔开。

**节点label实例操作**：

```bash
#查看所有节点的label
kubectl get nodes --show-labels

NAME               STATUS   ROLES                         AGE   VERSION 
k8s-master-node1   Ready    control-plane,master,worker   89d   v1.22.1 ta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostnams=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/mker=,node.kubernetes.io/exclude-from-external-load-balancers=
k8s-worker-node1   Ready    worker                        89d   v1.22.1 ta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostnams=linux,node-role.kubernetes.io/worker=

#查看某特定节点的label
kubectl get nodes k8s-master-node1 --show-labels

NAME               STATUS   ROLES                         AGE   VERSION 
k8s-master-node1   Ready    control-plane,master,worker   89d   v1.22.1 ta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostnams=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/mker=,node.kubernetes.io/exclude-from-external-load-balancers=

#给节点设置标签
kubectl label node node_name key=value

##例如
kubectl label node k8s-master-node1  123=456  #设置成功

kubectl get nodes k8s-master-node1 --show-labels | grep 123  #查询，结果略

#取消节点的某个标签
kubectl label node node_name key-

##举例
kubectl label node k8s-master-node1 123-     #取消key=value为123=456的label

#给所有节点设置标签
kubectl label node --all 123=456

kubectl get node --show-labels | grep 123   #查询，结果略

## 节点的标签栏里面有一项为roles，表示这个节点在集群中的角色，比如master，worker，control-plane(控制面板),etcd（管理存储数据库）或者为none（空）

## 决定roles值的label是 node-role.kubenetes.io/roles_name

## 这个键有没有值都无所谓，如果不设置值的话，value部分用""替代即可

# 给worker节点设置node-role.kubenetes.io标签
kubectl label nodes k8s-worker-node1 node-role.kubernetes.io/worker2=""

kubectl get node --show-labels k8s-worker-node1 
NAME               STATUS   ROLES            AGE   VERSION   LABELS
k8s-worker-node1   Ready    worker,worker2   89d   v1.22.1   123=456,beta.kubernetes.io/arch=amd64,   ##查询可知修改成功

# 取消办法与普通label一致,key- 方法
kubectl label nodes k8s-worker-node1 node-role.kubernetes.io/worker-

```



##### 创建在特定节点上运行的pod

​	在pod里通过nodeSelector 可以让pod在含有特定标签的节点上运行

案例实施：创建新pod，让其在有 123=456 的label的节点上运行

```yaml
kubectl run test110 --image=nginx:latest --dry-run=client --image-pull-policy=IfNotPresent -oyaml >> newPod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test110
  name: test110
spec:
  nodeSelector:
    diskxxx: ssdxx      #要确定集群中有含有这个标签的节点运行
  containers:
  - image: nginx:latest
    imagePullPolicy: IfNotPresent
    name: test110

kubectl apply -f newPod.yaml

kubectl get pod -owide
NAME      READY   STATUS    RESTARTS   AGE   IP           NODE               NOMINATED NODE   READINESS GATES
test110   1/1     Running   0          23s   10.244.0.7   k8s-master-node1   <none>           <none>

# 案例实施成功，test110成功运行在了有diskxxx=ssdxx标签的master节点上

## 方法二
## 指定nodeName

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test110
  name: test110
spec:
  nodeName: k8s-master-node1
  containers:
  - image: nginx:latest
    imagePullPolicy: IfNotPresent
    name: test110
```

##### Annotations 设置

不管是节点还是pod，或者其他对象，都还有一个属性Annotations，这个属性可以理解为注释





##### 节点的cordon 与 drain

​	如果想把某个节点设置为不可用的话，可以对节点实施cordan或drain操作，这样节点就会被标记为 SchedulingDisabled，新创建的pod就不会分配到这些节点上了

​	**节点的cordon**

​	如果某个节点要进行维护，希望此节点不再被分配pod，那么可以使用cordan把此节点标记为不可调用，但是运行在此节点上的pod依然会运行在此节点上。

 案例演示：

```yaml
##  创建一个deployment
kubectl create deployment nginx --image=nginx:latest -r 3 --image-pull-policy=IfNotPresent --dry-run=client  -o yaml >> d1.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - imagePullPolicy: IfNotPresent
        image: nginx:latest
        name: nginx
        resources: {}
status: {}

## 运行这个dp
kubectl apply -f d1.yaml

## 查看pod运行情况

## 查看各node情况

## 现在使用cordan命令将k8s-worker-node1 标记为不可用
kubectl cordan k8s-worker-ndoe1

## 查看node情况，发现被worker节点被标记为，SchedulingDisabled

## 这时修改dp.yaml 中的副本数 replicas为6 
kubectl scale deployment nginx --replicas=6

## 查看所有pod情况，发现新增的pod全在master节点上
## 这时如果删除原来运行在worker节点上的pod，那么所有节点都会运行在master上了（事关deployment，后面会解释）

##取消cordan状态
kubectl uncordan k8s-worker-node1
```

​	**节点的drain**

​	drain 与 cordan 都是暂时让节点不可用的功能，但是drain比cordan多了一个驱逐（evicted）效果，即当我们对某节点进行删除的时候，不仅会把此节点标记为不可用，同时还会把正在运行的pod删除,在节点离线之前，会把正在运行的pod迁移到其他节点上

```bash
## 将worker节点设置为不可用，drain方法
kubectl drain k8s-worker-node1

## 会报错，因为worker节点上运行了一些由daemonset控制的pod，此时依然在worker运行。所有需要引入 --ignore-daemonset 忽视由daemonset控制的pod

kubectl drain k8s-worker-node1 --ignore-daemnoset  --delete-local-data

## 会发现worker节点的pod已经开始删除.--delete-local-data: 这个标志也是可选的。如果指定了它，将强制删除与被排除节点上的Pod相关联的任何本地数据。本地数据包括与节点关联的 emptyDirs 或本地卷。请谨慎使用该标志，因为可能会导致数据丢失。

## 取消drain与取消cordan一样，都是uncordan
kubectl uncordan k8s-worker-node1

```

##### 节点taint及pod的tolerations

​	[节点亲和性](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) 是 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 的一种属性，它使 Pod 被吸引到一类特定的[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/) （这可能出于一种偏好，也可能是硬性要求）。 **污点（Taint）** 则相反——它使节点能够排斥一类特定的 Pod。

​     想象一下，某个公司有污点（假设是克扣工资），一般面试的人是不会选择这家公司的，如果有人可以容忍这个污点，那就可以来上班了。同理，如果一个pod能容忍节点上的污点，则就可以在此节点上运行

​	首先查看worker节点有没有污点

```bash
 kubectl describe nodes k8s-worker-node1 | grep -E '(Roles|Taints)'
## 命令解释：—E 是grep命令后跟正则表达式的意思，'(Roles|Taints)'是返回含有这两个关键字的行
Roles:              worker2
Taints:             <none>     ## 返回为none ，可以看到worker节点上没有污点设置、

```

​	为节点设置taint的语法如下：

```bash
kubectl taint node node_name key=value:effect   effect的值一般是NoSchedule
```

​	为所有节点设置taint

```bash
kubectl taint --all key=value:NoSchedule
```

注意：这里的value值是可以不写的，如果不写，语法是这样的

```bash
kubectl taint node k8s-worker-node1 key=:NoSchedule
```

删除taint

```bash
kuebctl taint nodes node_name key-  #如123-
```

如果需要pod在含有taint的节点上运行，则定义pod的时候就需要指定**toleration属性(容忍规则)**

在pod里定义toleration的格式如下

```yaml
tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  - key: "key2"
    operator: "Exists"
    effect: "NoExecute"
    
## operator 的值有以下两个
Eqoal: value需要和taint的value值一样
Exists: 可以不指定value的值
```

**设置operator的值为Equal**

​	在pod里定义tolerations的时候，如果operator的值为Equal的话，则value和effect的值要与节点的taint匹配才可以

 **operator的值为Exists的情况**

​     先说结论，在设置节点taint的时候，如果value的值非空，在pod里的tolerations字段只能写Equal不能写Exists。

举例value为空：

```bash
kubectl taint nodes k8s-worker-node1 123=:NoSchedule
```

#### label （资源标签）

lable是k8s系统中的一个重要概念,是一种分类，标识选择机制它的作用就是在资源上添加标识，来对资源进行区分和选择。

Label的特点：

* 以键值对 key/value的形式附加到各种对象上，如Node、Pod、Service等等

*  一个资源对象可以添加任意数量的Lable，同一个Lable也可以被添加到任意对象的资源对象上

* Label通常在资源对象定义时确定，当然也可以在对象创建后动态添加或者删除

  > 一些常用的Label如下:
  >
  > * 版本标签："version":"'release'',"version":"stable"
  > * 环境标签："environment":"dev","environment":"test"
  > * 架构标签："tier":"frontend","tier":"backed"

label selector （标签选择器） 用于查询和筛选拥有某些标签的资源对象

* 基于等式的label selector

  name = slave 选择所有包含lable中key="name"而value="slave"的对象

  env != production 选择所有包含lable中key="env"而 value不等于"production"的对象

* 基于集合的label selector

​		name in (master,slave) 选择所有包含lable中的key="name"且value="master"或"salve的对象

​		name not in (frontend) 选择所有包含lable中的key="name"且value不等于"frontend"的对象

标签的选择条件可以使用多个label selector进行组合，使用逗号","进行分隔即可。例如：

​		name = slave，env != production

​		name not in (frontend), env != production

##### 查看pod的标签

```shell
kubectl get pod pod_name  --show-labels

```



##### label的命名方式

```shell
#给Pod打标签,给test01打上了 key="version"而value="1"的标签
kubectl label pod test01 version=1
```



##### 如何更新label

```shell
kubectl label pod test01 version=2 --overwrite
```



##### 如何筛选label

```shell
kubectl get pod -l version=2.0

#筛选当前命名空间里label为version=2.0的pod
```



##### 如何删除label

```shell
kubectl get pod test01 test02 --show-labels   #先查询这pod的标签
NAME     READY   STATUS    RESTARTS      AGE     LABELS
test01   1/1     Running   1 (50m ago)   16h     run=test01,version=2.0
test02   1/1     Running   0             5m13s   run=test02,version=1.0

kubectl label pod test01 version-   #在key后面加一个 "-"就代表删去这个标签
pod/test01 labeled

kubectl get pod test01 test02 --show-labels #再查询发现test01的veriosn标签已经没有了
NAME     READY   STATUS    RESTARTS      AGE    LABELS
test01   1/1     Running   1 (53m ago)   16h    run=test01
test02   1/1     Running   0             8m9s   run=test02,version=1.0


```



###  存储管理

#### emptyDir卷挂载

​	emptyDir卷是一种临时存储，作用是在同一个pod里的不同容器之间共享数据

​	当删除pod时，emptyDir对应的目录会被一起删除，因为这种存储是临时性的，是以内存为介质的，并非是永久性的

​	本章所涉及的文件全部放在一个volume目录中

```bash
mkdir -p /volume ; cd /volume/
```

​	创建pod，按如下内容修改

```yaml
kubectl run pod pod1 --image=busybox:latest --dry-run=client -oyaml >> pod.yaml

apiVersion: v1
Kind: Pod
metadata:
  name: pod1
spec:
  volumes:
  - name: volume1
    emptyDir: {}
  - name: volume2
    emptyDir: {}
  containers:
  - name: c1
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']  #这里执行命令是因为busybox镜像运行时间很短，需要延长一些
    volumeMounts:
    - mountPath: /opt
      name: volume1
      
  - name: c2
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']
    volumeMounts:
    - mountPath: /opt
      name: volume1
      
# 这样就将c1和c2两个容器的 /opt/目录都挂载在 pod的emptyDir卷volume1上，这样就以这个空卷为中间介质，实现了两个容器的数据共享（c1的修改会同步到c2上，反之亦然）
```

使用空卷挂载有以下几个明显缺点：

* 数据不持久化，因为emptyDir是一个临时存储卷，当pod被删除时，里面存储的数据也会被删除

* 单点故障，emptyDir挂载在同一个节点上的所有Pod都共享相同的存储空间。如果节点发生故障或被重启，所有挂载了该emptyDir的Pod都会受到影响。

* 无法跨节点或跨集群共享数据，因为pod只在一个节点上运行

  

#### hostPath

​	这种卷类型允许将主机节点上的特定路径挂载到 Pod 中，使得 Pod 内的容器可以直接访问主机上的文件和目录。

使用 hostPath 挂载主机路径时，Pod 在调度到某个节点后，可以指定一个主机上的路径作为一个卷。然后，Pod 中的容器可以读取和写入该路径上的文件。这在某些情况下非常有用，例如要在容器中访问主机的日志文件、配置文件或其他非容器化的资源。

​	使用hotsPath类型的卷挂载yaml文件如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod2
  name: pod2
spec:
  volumes:
  - name: volume1
    hostPath: 
      path: /data
  containers:
  - image: busybox:latest
    name: c1
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 3000']
    volumeMounts:
    - mountPath: /opt
      name: volume1
  - image: busybox:latest
    name: c2
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 3000']
    volumeMounts:
    - mountPath: /opt
      name: volume1
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

​		在物理机的/data/下创建hhh并写入'hello'

```bash
cd /data/ ; touch hhh ; echo hello >> hhh
```

​		进入容器查看

```bash
kubectl   exec -it host-pod -- cat /opt/hhh
hello
```

​		在容器里的/opt/hhh中追加123

```bash
kubectl exec -it host-pod -- sh
echo 123 >> hhh
exit
```

​		查看物理机的/data/hhh目录

```bash
cat /data/hhh
hello
123
```

​		删除pod

```bash
kubectl delete pod host-pod --force
```

​		查看/data/hhh （挂载位置）

```bash
cat /data/hhh
hello
123
```

hostPath类型的卷挂载，虽然相比空卷挂载实现了删除pod后仍在本地保留数据的功能，但是因为其是直接挂载在主机的文件系统，如果容器的权限被改变，则很容易被入侵到敏感目录。所以hostPath的优缺点是：

* 简单易用：hostPath 是 Kubernetes 提供的一种卷类型，使用起来非常简单，无需额外配置。
* 高性能：hostPath 直接将主机节点上的文件系统路径挂载到容器中，没有额外的网络开销，可以获得较高的性能。
* 直接访问主机资源：使用 hostPath 可以直接访问主机节点上的文件或目录，适用于需要与主机节点进行直接交互的场景，如日志文件收集、共享配置等。

⚠️ hostPath 的缺点：

- 主机依赖性：hostPath 挂载的路径是主机节点上的路径，因此在不同节点上的容器可能无法访问相同的路径，这导致了主机依赖性。
- 可移植性差：由于 hostPath 的依赖性，当 Pod 需要在不同的节点之间迁移时，路径可能不存在或不匹配，降低了应用程序的可移植性。
- 安全性风险：使用 hostPath 可能会引入安全风险，因为容器可以直接访问主机的文件系统。如果容器被攻击或滥用权限，可能会访问到主机上的敏感文件。

#### NFS存储

​	不管是emptyDir还是hostPath存储，虽然数据都可以存储在服务器（主机）上，但是都没法同步到其他节点上，如果此前的pod出现了问题，deployment会新建一个新pod，如果这个pod仍在原来pod的节点上运行，那么就没有问题，但如果新建的pod在其他节点上，那就读取不到原来修改过的数据了

​	如果使用NFS存储（网络存储）的话，就不会有这个问题，NFS是一个网络文件存储系统，可以搭建一台NFS服务器，然后将Pod中的存储直接连接到NFS系统上，这样的话。无论Pod在节点上如何转移，只要Node跟NFS的对接没问题，数据就可以访问

首先需要搭建一个NFS服务器，这里为了简单，直接用master节点做nfs服务器

```bash
#先安装nfs安装包
yum install nfs-u* -y
##没有yum源的话可以用阿里云的 wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

#本地创建共享目录
mkdir /root/data/nfs -pv

#给共享目录暴露权限
vim /etc/exports    ##这是nfs默认读取的暴露目录
写入 /root/data/nfs   *(rw,sync,no_root_squash)

#设置开机自启动
systemctl enable nfs

##命令解释
  `*`：所有网段
  `rw`：读写模式；
- `all_squash`: 所有用户都会被压缩成为一个用户，默认情况下是nfsnobody（uid=65534）；
- `root_squash`: root用户也会被压缩成为nfsnobody（uid=65534）；
- `no_root_squash`: 不要对root用户进行压缩；
- `sync`：同步模式，数据在客户端和服务器之间传输时，如果发生错误，将导致NFS失败并返回错误信息给客户端；

```

创建一个使用了nfs存储类型的pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nfs-pod
  name: nfs-pod
spec:
  volumes:			##定义一个名为volume1的卷，类型为nfs，并指定nfs服务器的ip和本地挂载的目录地址
  - name: volume1
    nfs: 
      server: 192.168.120.99
      path: "/root/data/nfs"
  containers:
  - image: busybox:latest
    name: nfs-pod
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']   ##运行sleep 5000 的原因是busybox的容器在很快的运行完后就会终止
    volumeMounts:		## 在容器内定义卷挂载，卷名要与前面的卷名保持统一，并指定想要挂载的目录名称
    - name: volume1
      mountPath: /opt/
```

​		这样通过nfs服务器的挂载就实现了数据共享和持久化存储（只要nfs服务器不崩）

​		本质上，是把容器里的/xx目录挂载到192.168.120.99:/root/data/nfs上，如果xx不存在，则会自动创建

总结一下：

nfs类型的卷存储方式可以跨节点进行一种持久化存储，但是有以下几个问题：

* 性能瓶颈：NFS的性能可能受到网络带宽、延迟和服务器负载的限制
* 单点故障：如果NFS的服务器节点挂掉且只有这一个节点的话，则会影响到整个系统
* 安全性限制：NFS作为后端存储，因为每个人都会接触到后端存储，这样就带了安全隐患，因为要配置存储，必须要root用户，如果有人恶意删除或者拷贝其他人的存储数据就会很麻烦。



#### 持久性存储 PV 和 PVC

​		**PV**

​		Persistent Volume（持久性卷，简称pv）与指定后端存储关联，pv和后端存储都由专人来创建，pv不属于任何命名空间，全局可见。用户登录到自己的命名空间之后，只要创建pvc（Persistent Volume Claim，持久性卷声明）即可，pvc会自动和pv进行绑定。

​		示例演示：

```bash
# 查看本地现有pv
kubectl get pv
No resources found

# 用explain命令查看创建pv的yaml文件的资源清单
kubectl explain pv.spec
kubectl explain pv.spec.nfs      
kubectl explain pv.spec.capacity 
kubectl explain pv.spec.accessModes
kubectl explain pv.spec.storageClassName
kubectl explain pv.spec.persistentVolumeReclaimPolciy
kubectl explain pv.spec.volumeMode

# 配置一个pv

apiVersion: v1
Kind: PersistentVolume    ## 注意kind的k是小写
metadata:
  name: pv1
spec:
  nfs:					  ## 存储类型为nfs
    server: 192.168.120.99
    path: /data/nfs
  capacity:
    storage: 5Gi 
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle  #注意拼写
  
```

​		**pv的关键配置参数说明：**

* 存储类型 （nfs、cifs、clusterFS等）

    ```yaml
    nfs:
      server: nfs服务器ip
      path: nfs服务器存储目录
    ```

    

* capacity （存储能力）

    目前只支持存储空间大小的设置 

    ```yaml
    capacity: 
      storage: 5Gi
    ```

    

* accessModes （访问模式）

    用于管理用户对存储资源的访问权限，包括以下几种方法：

    * ReadWriteOnce (RWO): 读写权限，但是只能被单个节点挂载
    * ReadOnlyMany (ROX): 只读权限，但是可以被多个节点挂载
    * ReadWriteOnce (RWX): 读写权限，但是可以被多个节点挂载

    <u>值得注意的是不同的存储类型可能支持的访问模式也不同</u>

* persistentVolumeReclaimPolicy (回收策略)

    当pv不再使用之后，对其有三种处理策略：

    * Retain （保留），保留数据，需要管理员手动清理数据

    * Recycle （回收），清除pv中的数据，效果相当于执行 rm -rf /volume/*

    * Delete   （删除），后端存储 （NFS服务器）完成volume的删除操作，常见于云服务的存储服务

        <u>底层存储类型不同，有时候回收策略也不同</u>		

* status （pv状态）

    一个pv的生命周期中可能会出现4种不同的状态：

    * Available（可用）：表示可用状态，但还未被任何pvc绑定
    * Bound（已绑定）：表示pv已经被pvc绑定
    * Released（已释放）：表示pvc被删除，但是资源还未被集群重新声明
    * Failed （失败）：表示该pv的自动回收失败
    
*  volumeMode (卷模式)

    指定了卷的使用模式，即如何使用卷中的数据。

    * Filesystem：表示卷被挂载为一个文件系统，并可用在容器内的路径上访问。这是最常见的卷模式，使容器可以像操作本地文件系统一样操作卷中的文件
    * Block： 表示卷以块设备的形式被挂载，容器可以直接访问该块设备。这种模式适用于需要直接访问块设备的情况，比如数据库存储等

​	**PVC**

​	PVC是基于命名空间创建的，不同命名空间里的pvc相互隔离

​	pvc通过**storage的大小和accessModes的值与pv进行绑定**，即如果pvc里storage的大小、accessModes的值和pv里storage的大小、accessModes的值都一样的话，那么pvc会自动和pv进行绑定。

​	 创建pvc：

```yaml
vim pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: PVC1
  namespace: xxx
spec:
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests: 
      storage: 5Gi
      
## accessMmodes、volumMode、resources的值都与pv1一样，所有pvc创建成功后会自动绑定
```

​		在pod中指定pvc：

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
  namespace: demo
spec:
  volumes:
  - name: myv
    persistentVolumeClaim: 
      claimName: pvc1       ## 指定pvc为上一个创建的pvc1
  containers:
  - image: busybox:latest
    imagePullPolicy: IfNotPresent
    name: pod1
    volumeMounts:
      - mountPath: "/mnt"
        name: myv 
    command: ['sh','-c','sleep 2000'] 
```

​	值得注意的是：pv和pvc的accessModes值相同的情况下，如果pv的的storage大小大于等于pvc的storage那么pvc可以成功绑定pvc，如果pv的storage小于pvc的storage，那pv与pvc不能绑定

**Storage Class Name** （存储类名）

​	有一个问题，如果pv1只能与一个pvc绑定的话，那如果pv1满足了多个pvc的条件，那么哪个pvc能和pv1绑定呢？

​	如果要控制哪个pvc能和pv1绑定，可以通过定义storageClassName进行控制

```yaml
# 创建有storageClassName的pv
vim pv01.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce  ## 注意这里要空两格再 - 
  volumeMode: Filesystem   ## F要大写
  storageClassName: xxx
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 192.168.120.99
    path: /data/nfs

# 创建有storageClassName的pvc
vim pvc01.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc01
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: xxx
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi


## 查看pv01与pvc01是否绑定
 kubectl get pvc -owide -n demo
NAME    STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE     VOLUMEMODE
pvc01   Bound     pv01     5Gi        RWO            xxx            12s     Filesystem

status为Bound，绑定成功

```

注： 如果存储类名不一样，那就无法绑定，如果不指定存储类目，在众多pvc条件相同的情况下不能确定是哪一个可以成功绑定

​	总结：

​			pv和pvc的出现让持久化存储变得更加安全

#### 动态卷供应

​	如果在不同的ns里要同时创建不同的pvc的话，需要提前创建好pv，然后才能为pvc提供存储，这种方法太过于麻烦，所以可以通过storageClass（简称sc）来解决这个问题

​	最终的效果是管理员不需要提前创建pv，只要创建好storageClass之后就不用管pv了，用户创建pvc的时候，storageClass会自动创建一个pv和pvc来进行绑定

##### storageClass 的工作流程

​	定义storageClass时必须要包含一个分配器（provisioner）,不同的分配器指定了动态创建pv时使用什么后端存储。先看下面的三个例子：

第一个例子： 分配器使用aws的ebs作为pv的后端存储

```yaml
apiVersion: v1
kind: StorageClass
metadata: 
  name: slow
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```

第二个例子：分配器使用lvm作为pv的后端存储 （lvm是一种卷管理工具）

```yaml
apiVersion: v1
kind: StorageClass
metadata: 
  name: sci-lvm
provisioner: lvmplugin.csi.alibabacloud.com
parameters:
    vgName: volumegroup1
    fsType: ext4
reclaimPolicy: Delet
```

第三个例子，使用hostPath作为pv的后端存储

```yaml
apiVersion: v1
kind: StorageClass
metadata: 
  name: csi-hostpath-sc
provisioner: hostpath.csi.k8s.io
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: tr
```

##### 利用nfs创建动态卷存储



##### 部署nfs分配器



##### 部署storageClass



### 密码管理

​	在创建pod的时候不少情况下是需要密码的，比如使用mysql镜像需要配MYSQL_ROOT_PASSWORD，使用wordpress镜像需要使用WORDPRESS_DB_PASSWORD来指定密码等

​	如果直接把密码信息保存在创建的pod的yaml文件，创建好pod之后，通过kubectl describe pod pod name 就很容易看到我们所设置的密码，这里存在一定的安全隐患。

​	我们可以用一个东西专门来存储密码，这个东西就是secret及configmap。

##### secret

secret的主要作用是存储密码信息，以及往pod里传递文件。secret以键值对的方式存储，格式如下：

```bash
键=值 或 key=value
```

##### 创建secret

​	创建secret的方法有很多，可以直接指定key和value，也可以把一个文件的内容作为value，还可以直接写yaml文件，下面分别用不同的方法来创建

​	**命令行创建secret**

```bash
kubectl create secret generic secret-name --from-literal=key1=value1 ...

# 以通用（generic）方式创建一个secret，并设置一个键值对为内容

kubectl describe secrets secret-name 

Name:         secret-name
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data			# 可以看到有一个数据，将值隐藏，只显示为6个字节
====
key1:  6 bytes

```

​	**查看sercet的键值对**

```bash
kubectl get secret secret-name -oyaml

kubectl get secret secret-name -oyaml
apiVersion: v1
data:
  key1: dmFsdWUx    # 可以看到这里的值被重新用base64编码了

```

​	**解码**

```bash
echo "dmFsdWUx" | base64 --decode

value1[root@k8s-master-node1 ~]# 

```

​	**把文件创建为secret**

也可以把一个文件创建为secret，此时文件的文件名叫做key，文件的内容为value，如果要把一个文件创建为secret的话，使用的命令如下。

```bash
#命令示例 kubectl create secret generic mysecret2 --from-file=file1 --from-file=file2

# 查看要创建为secret的文件
cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
## kubeeasy managed start
192.168.120.99 apiserver.cluster.local
192.168.120.99 k8s-master-node1
192.168.120.110 k8s-worker-node1
## kubeeasy managed end
文件名为hosts，现在把这个文件创建为一个secret

kubectl create secret generic mysecret2 --from-file=/opt/hosts

kubectl get secret mysecret2 -oyaml
apiVersion: v1
data:
  hosts: MTI3LjAuMC4xICAgbG9jYWxob3N0IGxvY2FsaG9zdC5sb2NhbGRvbWFpbiBsb2NhbGhvc3Q0IGxvY2FsaG9zdDQubG9jYWxkb21haW40Cjo6MSAgICAgICAgIGxvY2FsaG9zdCBsb2NhbGhvc3QubG9jYWxkb21haW4gbG9jYWxob3N0NiBsb2NhbGhvc3Q2LmxvY2FsZG9tYWluNgojIyBrdWJlZWFzeSBtYW5hZ2VkIHN0YXJ0CjE5Mi4xNjguMTIwLjk5IGFwaXNlcnZlci5jbHVzdGVyLmxvY2FsCjE5Mi4xNjguMTIwLjk5IGs4cy1tYXN0ZXItbm9kZTEKMTkyLjE2OC4xMjAuMTEwIGs4cy13b3JrZXItbm9kZTEKIyMga3ViZWVhc3kgbWFuYWdlZCBlbmQK
kind: Secret
## 发现值被重编码了

## 使用base64解码
kubectl get secret mysecret2 -o jsonpath='{.data.hosts}' | base64 -d



```



**yaml文件形式创建secret**

下面的例子会创建一个名为mysecrt5的secret，里面包含两个键值对： xx=tom和yy=redhat

```yaml
# 先将xx=tom 和 yy=redhat 用base64重编码
echo 'tom' | base64 
dG9tCg==

echo 'redhat' | base64 
cmVkaGF0Cg==

# 创建yaml文件
vim secret5.ayml

apiVersion: v1
kind: Secret
metadata: 
  name: mysecret5
type: Opaque
data: 
  xx: dG9tCg==
  yy: cmVkaGF0Cg==

```

##### 使用secret

​	上面已经把secret创建出来了，那么如何在pod里使用这些sercet呢？主要有两种方式来引用卷，即卷和变量的方式

​	**方法一： 以卷的方式**

​    下面创建一个名字为pod1的pod，以卷的方式挂载mysecret2到容器的/etc/test 目录里

```yaml
vim pod1.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: nginx:latest
    name: pod1
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: xx
      mountPath: "/etc/test"
  volumes:
  - name: xx
    secret:
      secretName: mysecret2

kubectl apply  -f  pod1.yaml 
kubectl get pod -owide

 kubectl exec pod1 -it -- bash
root@pod1:/# cat /etc/test/hosts 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
## kubeeasy managed start
192.168.120.99 apiserver.cluster.local
192.168.120.99 k8s-master-node1
192.168.120.110 k8s-worker-node1
## kubeeasy managed end

## 在这个yaml文件里创建了一个名字为xx，类型为secret的卷，使用mysecret2，在pod1里的容器里把xx这个卷挂载到/etc/test目录里，因为nysecret2有一个键hosts，值是文件/etc/hosts的内容，所以容器/etc/test/目录里有一个文件hosts，内容就是/etc/hosts的内容。


```

此时，有一个问题，如果创建的secret中有两个文件 ,那不想把两个文件都写入挂载点，只想使用一个文件的话，那怎么办？

> 例如 ：kubectl create secret myscret generic  --from-file=xxx --from-file=xxx

此时可以用subPath来解决这个问题

修改pod1.yaml 如下

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: nginx:latest
    name: pod1
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: xx
      mountPath: "/etc/test/issue"
      subPath: issue    ##若需要指定从secret里引入哪个文件，就使用subPath
  volumes:
  - name: xx
    secret:
      secretName: mysecret2、
      
## subPath指定从secret里引入哪个文件，把这个文件写入到/etc/test/里，然后命名为issue。如果想命名为其他名字，比如filex，则mountPath的值应该写成/etc/test/filex。如果mountPath的值只写为/etc/test/的话，则是把issue写入/etc/里，然后命名为test。即/etc/test/是一个文件，内容为issue的内。
```

​		**方法二：以变量的方式**

前面讲了，在定义pod的yaml文件的时候，如果想使用变量的话，格式为：

```yaml
env:
- name: 变量名
  value: 值
```

​	但是如果想从secret引入值得话，此处就不用写value了，而是写valueFrom，即从什么地方来引用这个变量值。如下所示：

```yaml
env:
- name: 变量名
  valueFrom: 
    secretKeyRef:
      name: secretX
      key: keyX
# 意思是，变量d值会使用secretX这个secret里得keyX这个键对应的值
```

具体解释：

  `valueFrom` 结合 `secretKeyRef`，可以用于从 Kubernetes Secret 对象中提取值，并将其注入到 Kubernetes 资源配置的特定位置。 使用方法如下：

 1️⃣ `secretKeyRef` 指定了对 Secret 对象的引用。 

2️⃣ 在 `secretKeyRef` 中，需要提供包含要提取值的 Secret 的 `name`。

 3️⃣ 另外，还需要指定 Secret 中保存所需值的 `key`。

 通过使用 `valueFrom` 和 `secretKeyRef`，您可以安全地访问存储在 Kubernetes Secrets 中的敏感信息，例如密码、API 密钥或其他机密数据，而无需直接在资源配置中公开它们。 这是一个方便的功能，可以确保 Kubernetes 部署中敏感数据的安全性和分离！

​	mysql示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: mysql:latest
    name: pod1
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret1
          key: yy
~                      
```

##### configmap

​	configmap 简称cm，其作用与secret一样，主要是可以存储密码或者往pod里传递文件。configmap也可以以键值对的方式来存储数据。

​	configmap可以用多种方式来创建。secret和configmap的主要区别是，secret使用了base64进行编码，而configmap不需要



查看现有cm

```bash
kubectl get cm
```

**创建configmap**

​	创建cm的办法有很多，可以直接指定key和value，也可以把一个文件的内容作为value，还可以用yaml文件

​	**命令行方式创建**

```bash
 kubectl create configmap mycm --from-literal=k1=123 --from-literal=k2=456
configmap/mycm created

# 查看一下cm详情
kubectl describe cm mycm
Name:         mycm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
k1:
----
123
k2:
----
456
Events:  <none>

#这里在data字段下就可以看到k1和k2的值
```

​	**以文件内容的方式创建**

```bash
kubectl create configmap mycm2 --from-file=/etc/hosts --from-file=/etc/issue
```

**使用configmap**

​	**以卷的方式使用**

```yaml
# 先创建一个有内容的configmap
比如mycm3

# 创建pod如下
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  replicas: 1
  selector:
    matchLabels:
      run: pod1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: pod1
   spec:
      volumes:          ##创建卷，类型为configmap，指定对应的cm
      - name: xx
        configMap:
          name: mycm3

      containers:
      - image: nginx:latest
        name: pod1
        imagePullPolicy: IfNotPresent
        volumeMounts:   ##容器内创建卷挂载，卷名要与上面的卷名一样，指定容器内挂载路径
        - name: xx
          mountPath: "/etc/test" 
          
## 进入容器，查看对应挂载目录

```



### Deployment （pod控制器）

​	前面我们直接创建出来的pod都是不稳定的，不健壮的，挂掉之后就不会自动起来 ，这样运行在容器里的应用也就无法正常运行了，我们可以利用deployment来提升健壮性

​	在k8s中，pod是最小的调度单位，但是k8s不去直接管理pod，而是通过pod控制器来管理pod，确保pod资源符合预期状态，当pod的资源出现故障时，会尝试进行重启或重建pod，**deployment就是一种pod控制器**

​     deployment（简称deploy） 是一个控制器，只要告诉deployment需要几个pod，如下图是三个，那deploy就会始终保持有3个pod，如果pod1挂掉了，则deployment会马上生成一个新pod，保证环境里有三个pod。

​	![image-20230925094112906](C:\Users\Blue Bridge\AppData\Roaming\Typora\typora-user-images\image-20230925094112906.png)

#### 创建和删除 deployment

​	建议通过yaml文件的方式来创建deployment，因为从k8s v1.18.x开始，在命令行里基本上除了 --image 外，不再支持其他参数。

**用命令行生成deployment的yaml文件:**

```shell
kubectl create deployment test1 --image=nginx --dry-run=clienlst -o yaml > d1.yaml

#模拟创建一个以nginx为镜像的名为test1的deployment，并以yaml文件重定义，但不真的生成资源（pod，deploy）

cat d1.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test1    ##deploy的标签可以和pod的标签不一致
  name: test1     ##deploy的名字
spec:
  replicas: 3     ##pod数量
  selector:
    matchLabels:
      app: test1  ##这个app的key=value与下面的要保持一致
  strategy: {}
  template:        #pod的设置
    metadata:
      creationTimestamp: null
      labels:
        app: test1 ##   这两个值要保持一致
    spec:
      containers:
      - image: nginx
        imagePullPolicy: IfNotPresent  #镜像拉取策略
        name: nginx
        resources: {}
status: {}


```



**删除一个pod，验证是否会自动创建新的pod**

```shell
kubectl get pods  #查看当前pod

NAME                     READY   STATUS    RESTARTS   AGE
test1-78747d7d6c-8mbzr   1/1     Running   0          9s
test1-78747d7d6c-ld4fk   1/1     Running   0          9s
test1-78747d7d6c-x99rk   1/1     Running   0          9s


kubectl delete pod test1-78747d7d6c-8mbzr ##删除pod

pod "test1-78747d7d6c-8mbzr" deleted

kubectl get pods -owide   ##继续查看，发现deployment又创建了一个新pod补上
NAME                     READY   STATUS    RESTARTS   AGE   IP             NODE               NOMINATED NODE   READINESS GATES
test1-78747d7d6c-5lbh2   1/1     Running   0          4s    10.244.1.108   k8s-worker-node1   <none>           <none>
test1-78747d7d6c-ld4fk   1/1     Running   0          72s   10.244.0.61    k8s-master-node1   <none>           <none>
test1-78747d7d6c-x99rk   1/1     Running   0          72s   10.244.1.107   k8s-worker-node1   <none>           <none>


#因为在deploy的配置文件里定义了 replicas为3，所以为了保持3个pod，删除一个之后，deployment会立即生成一个新的pod。
```



##### 查看deployment的详细信息

```shell
kubectl get deployments.apps 

NAME    READY   UP-TO-DATE   AVAILABLE   AGE
test1   3/3     3            3           21m

#READY 3/3 表示有三个pod可用
#AVAILABLE 3 也是表示三个pod可用
```



##### 删除deployment

```shell
kubectl delete deployments.apps test1 -n deployns

kubectl get pods -n deployns   #再查看dep创建的pod，正在删除中
NAME                     READY   STATUS        RESTARTS   AGE
test1-6d5588f8cc-fxlhb   0/1     Terminating   0          2m42s


#比赛环境下无法切换ns，尽早熟悉-n写法
```

##### deployment 健壮性测试

​	已知deployment在一个pod被删除的情况下会自动创建于一个新pod以保证稳定性，如果该节点关机了呢？原来运行在此节点的pod怎么办

​	答案是：pod会在其他节点上去运行，在故障的几分钟内，master仍会等待pod的恢复，若等几分钟还没恢复，会执行删除，删除完毕后，master会重新调度新pod代替。但是原节点处于关机状态，master无法和原节点通信，所以原节点的pod就会处于失联状态，才会看到删除的两个pod状态为Terminating

##### 修改deployment

​	通过命令行修改

```bash
kubectl scale deployment --replicas=2 deploy_name
```

​	通过kubecl edit 命令在线修改deployment

```bash
kubectl edit deployment pod1 

spec:
  progressDeadlineSeconds: 600
  replicas: 4    ## 直接修改副本数
  revisionHistoryLimit: 10
 
## 值得注意的是通过edit命令直接修改的操作不会同步到源yaml文件里，修改后的配置会直接应用到 Kubernetes 集群中的相应资源上，而不会自动更新 YAML 文件。
```

​	通过修改yaml文件方式

```bash
vi pod1.yaml

spec:
  replicas: 1     ## 直接进行修改
  selector:
    matchLabels:
      run: pod1

kubectl apply -f pod1.yaml  ## 更新修改
```



##### 水平自动更新HPA

​	如果老是手动修改deployment的副本数，那在高负载场景下会力不从心，所以我们需要k8s根据负载情况自动的调节需要多少pod，这就是水平自动更新HPA

​	HPA通过检测pod的cpu负载通知depolyment，让其更新pod的数量以减轻单一pod的负载

**配置HPA**

​	可以直接用命令的方式创建HPA

```bash
kubectl autoscale deployment name --min=最小副本数  --max=最大副本数 --cpu-percent=X

## --min 是设置deploy的最小运行的副本数，--max是最多有几个
## --cpu-percent 是确保pod内的cpu占用不超过百分之X，如果超过了就扩展副本数，最大扩展到--max的数量
```

​	查看HPA

```bash
kubectl  get hpa

NAME   REFERENCE         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
pod1   Deployment/pod1   <unknown>/70%   2         5         3          16h

```













##### 静态 Pod （static Pod）

静态Pod直接有特定的节点上的kubelet进程来管理，不通过master节点上的apiserver。无法与我们常用的控制器Deployment或DaemonSet进行关联，它由Kubelet进程自己监控，当pod崩溃时重启该pod，kubelete也无法对它们进行健康检查。静态Pod始终绑定在某一个kubelet，并且始终运行在同一个节点上。 

> Kubelet会自动为每一个静态Pod在k8s的apiserver上创建一个镜像Pod（mirror Pod），因此我们可以在apiserver中查询到该Pod，但是不能通过apiserver进行控制（例如删除）

创建静态Pod有两种方式：配置文件和HTTP两种方式

##### Pod hook

K8s为我们的容器提供了生命周期钩子，也就是Pod hook。

Pod hook  允许在Pod的生命周期中执行自定义操作

Pod hook  可分为两类：**初始化钩子和终止钩子**

* 初始化钩子可以定义在Pod启动过程中的特定时间点触发的操作，比如在容器 *启动之前或启动之后执行某些任务*
* 终止化钩子可以定义在Pod终止（被删除）之前触发的操作，例如保存状态，发送不可恢复的事件等等。

**两种容器生命周期钩子函数：**

* postStart：当容器创建后立即执行的钩子函数。可用于启动后台进程，注册服务或执行其他初始化操作。

需要注意的是，如果钩子花费时间太长以至于不能运行或者挂起，容器将不能达到 running 状态

* preStop：当容器终止之前执行的钩子函数。容器终止之前，k8s会发送一个终止信号给容器，并等待一段时间来让钩子函数执行清理操作，如保存数据，关闭连接等。

需要注意的是，这个钩子是同步的，必须在删除容器的调用发出之前完成，主要用于优雅关闭应用程序;如果钩子在执行期间挂起，Pod阶段将停留在running状态并且永远也不会达到failed状态

*要使用 Pod hook 需在 Pod 的 YAML 文件中 定义相关的钩子函数和调用时机。*这样，当kubenetes运行时，就会自动触发这些函数，以执行预定义的操作。

**有两种方法来实现钩子函数**

* Exec - 用于执行一段特定的命令，不过要注意的的是该命令消耗的资源会被计入容器。
* HTTP-对容器上的待定的端点执行HTTP请求。

**postStat 应用**

```yaml
---     ##如果有一文件中有多个YAML则用---分隔开来
apiVersion: v1
kind: Pod
metadata:
  name: hook-demo1
  labels: 
    app: hook
spec:
  containers:
  - name: hook-demo1
    image: nginx
    ports:
    - name: webport
      containerPort: 80
    lifecycle:         ##生命周期
      postStart:
        exec:
          command: ["/bin/sh","-c","echo Hello from the postStart Handler > /usr/share/message"]  ## []的形式是list的第二种写法，即每个命令不另起一行，而用大括号包裹起来中间用逗号隔开。
          
          
 ##进入容器
 kubectl exec hook-demo1 -it /bin/bash
 #-it进入容器后提供一个交互式终端
 
 ##查看字符串写入是否成功
 cat  /usr/share/message 
```



#### k8s explain 解释 命令

```shell
#查看关于pod的解释
kubectl explain pod

#查看关于
kubectl explain pod.metad
```

