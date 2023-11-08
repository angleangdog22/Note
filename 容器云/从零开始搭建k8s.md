[TOC]

## 从零开始搭建k8s

一、前言
本文讲述从零开始搭建k8s集群，均使用国内镜像，版本均统一，使用两个虚拟机，一个主节点，一个从节点，保证k8s一次搭建成功。



注意：Kubernetes，简称K8s，是用8代替名字中间的8个字符“ubernete”而成的缩写。



二、Centos最小化安装
安装K8S需要使用Centos7，每个机器至少2个处理器和2G，这是k8s官网是要求的。



2.1 vmware最小化安装centos7
vmware最小化安装centos7，下载好镜像，不断下一步，直到完成。

先到官网 https://www.centos.org/download/ 下载centos.iso，如下：



国内的，直接选阿里云镜像就好 http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/

具体安装可以参见

其实没这么麻烦，只需要配置磁盘这个必选就好了，其他都不用配置，不断下一步，最好就安装好了。

2.2 配置静态IP
创建好的虚拟机 ping www.baidu.com 失败，需要配置好静态ip，才能连同外网



### K8S集群搭建

> 从3.1到3.6，所有机器都执行
> 3.7和3.8只在主节点上执行
> 3.9的kube join只在从节点上执行(测试kubectl get nodes在主节点上执行，因为从节点没有.kube/，无法通过kubectl操作k8s集群)
> 3.10集群新建pod测试，在主节点上执行，因为从节点没有.kube/，无法通过kubectl操作k8s集群

#### 3.1 更新yum
3台机器都需要执行

```bash
yum -y update
yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp
```

#### 3.2 安装Docker
在每一台机器上都安装好Docker，版本为18.09.0

##### 01 安装必要的依赖

```bash
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
```


#### 02 设置docker仓库

```bash
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

然后设置一下阿里云镜像加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://orptaaqe.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
```


#### 03 安装docker

```bash
yum install -y docker-ce-18.09.0 docker-ce-cli-18.09.0 containerd.io
```

#### 04 启动docker

```bash
sudo systemctl start docker && sudo systemctl enable docker
```

#### 05 检查docker是否安装并启动好了

```bash
ps -ef|grep docker
```

### 3.3 修改hosts文件
#### 01 机器192.168.100.150执行两句

```bash
sudo hostnamectl set-hostname xx
```

```bash
vi /etc/hosts
192.168.100.150 xx
192.168.100.151 xx
```


#### 02 机器192.168.100.151执行两句

```bash
sudo hostnamectl set-hostname xx
```

```bash
vi /etc/hosts
192.168.100.150 m
192.168.100.151 w1
```

#### 03 每天机器上都要测试

```bash
ping m  
ping w1
```


每个步骤配置完成之后，都要局部测试，这样才不会出问题



### 3.4 系统基础前提配置
每个机器上都要执行（机器192.168.100.150 和 机器192.168.100.151），如下：

##### (1)关闭防火墙

```
systemctl stop firewalld && systemctl disable firewalld
```



##### (2)关闭selinux

```
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```



#####  (3)关闭swap

```
swapoff -a
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
```



##### (4)配置iptables的ACCEPT规则

```
iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
```



##### (5)设置系统参数

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```
sysctl --system
```


### 3.5 安装 kubeadm, kubelet and kubectl
每个机器上都要执行（机器192.168.100.150 和 机器192.168.100.151），如下：

#### (1)配置yum源

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

#### (2)安装kubeadm&kubelet&kubectl

```
yum install -y kubectl-1.14.0-0
yum install -y kubelet-1.14.0-0
yum install -y kubeadm-1.14.0-0
```

这三个东西一定要保证版本统一，一定要安装这个安装顺序来完成

因为kubeadm依赖kubelet，如果先安装kubeadm，则kubelet的1.23版本就会被安装上去，然后再执行 yum install -y kubelet-1.14.0-0 会安装不上去，然后版本不统一，主节点初始化init报错



及时测试：

```
kubectl version
kubelet --version
kubeadm version
版本必须都是 1.14.0，如果版本错了要删除
```

```bash
# 删除命令
yum -y remove kubelet
yum -y remove kubectl 
yum -y remove kubeadm
```

#### (3) docker和k8s设置同一个cgroup

 编辑docker的daemon.json文件,每个节点都要执行

```
vi /etc/docker/daemon.json
    "exec-opts": ["native.cgroupdriver=systemd"],
    
```

```
systemctl restart docker
```

​    

kubelet，这边如果发现输出directory not exist，也说明是没问题的，大家继续往下进行即可

```
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 
```

```
systemctl enable kubelet && systemctl start kubelet
```


### 3.6 配置国内镜像
#### (1) 创建kubeadm.sh脚本，用于拉取镜像/打tag/删除原有镜像

```sh
#!/bin/bash

set -e

KUBE_VERSION=v1.14.0
KUBE_PAUSE_VERSION=3.1
ETCD_VERSION=3.3.10
CORE_DNS_VERSION=1.3.1

GCR_URL=k8s.gcr.io
ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/google_containers

images=(kube-proxy:${KUBE_VERSION}
kube-scheduler:${KUBE_VERSION}
kube-controller-manager:${KUBE_VERSION}
kube-apiserver:${KUBE_VERSION}
pause:${KUBE_PAUSE_VERSION}
etcd:${ETCD_VERSION}
coredns:${CORE_DNS_VERSION})

for imageName in ${images[@]} ; do
  docker pull $ALIYUN_URL/$imageName
  docker tag  $ALIYUN_URL/$imageName $GCR_URL/$imageName
  docker rmi $ALIYUN_URL/$imageName
done
```


#### (2) 运行脚本和查看镜像

 运行脚本

```
sh ./kubeadm.sh
```

查看镜像

```
docker images
```


至此，上面的3.1到3.6是每个节点都要执行的，就是机器 192.168.100.150 和 192.168.100.151 都要同时执行的



### 3.7 kube init初始化master
到此为止，两个机器 192.168.100.150 和 192.168.100.151 还需要没有主次之分，还没有说哪个是k8s的主节点，现在 3.7 这个步骤只需要在 192.168.100.150 节点执行，让这个机器成为k8s主节点。

#### (2)初始化master节点

```
kubeadm init --kubernetes-version=1.14.0 --apiserver-advertise-address=47.113.192.113 --pod-network-cidr=10.244.0.0/16
```

在 192.168.100.150 机器上执行，然后这个 --apiserver-advertise-address=192.168.100.150 这样写

会得到一句

```
kubeadm join 192.168.100.150:6443 --token 7oahxf.bk0rywevgu0xjouy

–discovery-token-ca-cert-hash sha256:4f84a48cc41343848d0b2f807c38937bc96d3d43e821f06b0c143a5282a38204
```

一定要复制粘贴出来，下一个步骤在 192.168.100.151 机器上执行这句。（每个人的不一样）

初始化完成执行后，继续只在 主节点 上执行，这三句，如下：

```
mkdir -p $HOME/.kube
cd .kube/
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


注意：我们所说的只有主节点才能通过 kubectl 命令操作k8s集群是因为，只有主节点上才完成这三步操作，才有 .kube 目录，如果从节点上也完成了这三句，也有 .kube 目录，也是可以通过 kubectl 命令操作k8s集群的。



#### (4) 测试：验证pod和健康检查（这两句也是只在master节点上执行）

验证pod

```
kubectl get pods -n kube-system
```

健康检查（不要怀疑，就是healthz）

```
curl -k https://localhost:6443/healthz
```


### 3.8 部署calico网络插件
k8s中，网络插件是为了k8s集群内部通信，所以必须安装一个网络插件，有很多选择，这里选择calico，同样在master节点上操作

在k8s中安装calico（这条命令很快的）

```
kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
```



确认一下calico是否安装成功  -w可以实时变化（看到calico都好了表示网络插件好了）

```
kubectl get pods --all-namespaces -w
```

测试验证是否安装成功，这里的calico两个显示Running就表示安装成功了。



从3.1到3.6，所有机器都执行，3.7和3.8只在主节点上执行，接下来的3.9的kube join只在从节点上执行(测试kubectl get nodes在主节点上执行，因为从节点没有.kube/，无法通过kubectl操作k8s集群)



### 3.9 kube join
#### (1) 在从节点上执行kube join命令，如下

```
kubeadm join 192.168.100.150:6443 --token 7oahxf.bk0rywevgu0xjouy \
    --discovery-token-ca-cert-hash sha256:4f84a48cc41343848d0b2f807c38937bc96d3d43e821f06b0c143a5282a38204
```


记得保存初始化master节点的最后打印信息，注意这边大家要自己的，上面我的只是一个参考



#### (2)在master节点上检查集群信息（node的正常状态是ready,pod的正常状态是running）

```
kubectl get nodes
```


当每个节点都是Ready，表示完成。

### 3.10 新建Pod测试整个集群
#### (1)定义pod.yml文件，比如pod_nginx_rs.yaml

```yaml
cat > pod_nginx_rs.yaml <<EOF
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: nginx
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
EOF
```


#### (2)根据pod_nginx_rs.yml文件创建pod

```
kubectl apply -f pod_nginx_rs.yaml
```

#### (3)查看pod

```
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx
```


#### (4)感受通过rs将pod扩容

```
kubectl scale rs nginx --replicas=5
kubectl get pods -o wide
```


#### (5)删除pod

```
kubectl delete -f pod_nginx_rs.yaml
```

#### （6）命令补全

这样安装的kubelet是没有命令补全的

```
yum install bash-completion
source /usr/share/bash-completion/bash_completion

echo 'source <(kubectl completion bash)' >>~/.bashrc
source ~/.bashrc
```



## 四、尾声

从零开始搭建k8s集群，完成了。

天天打码，天天进步！！
-----------------------------------
