[TOC]

## 使用Jenkins Slave构建CICD



​	Jenkins 是目前最受欢迎的开源CI/CD工具，使用Java语言编写，用于持续集成，集成了所有类型的开发生命周期流程，包括构建、文档、测试、打包、阶段、部署、静态分析等。Jenkins拥有超过1000个插件，可以轻松配置和自定义任何项目。

​	在Kubernetes集群中使用Jenkins工具进行持续集成与发布的方案如图1所示。

![none](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/c3612e84a493cefb7e3001dd99e8ed3f/image001.png)

#### 2.1 部署Jenkins

​	（1）基础准备

​		将软件包Jenkins-Slave.tar.gz下载到本地。

```shell
[root@master ~]# curl -O http://mirrors.douxuedu.com/competition/Jenkins-Slave.tar.gz
```

​		解压软件包并导入镜像：

```shell
[root@master ~]# tar -zxvf Jenkins-Slave.tar.gz
[root@master ~]# ctr -n k8s.io image import Jenkins-Slave/images/images.tar
```

​		集群所有节点修改Containerd配置，添加Harbor仓库的认证信息：

```shell
# vi /etc/containerd/config.toml
……
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]  # 此字段后加入以下配置
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."harbor.com"]
          endpoint = ["http://harbor.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."10.24.2.200"]
          endpoint = ["http://10.24.2.200"]
……
```

​		重启Containerd以生效配置：

```shell
# systemctl daemon-reload 
# systemctl restart containerd
```

​		将镜像推送到Harbor仓库中：

```shell
[root@master ~]# ctr -n k8s.io images tag docker.io/library/maven:3.8.2-openjdk-8 10.24.2.200/library/maven:3.8.2-openjdk-8
[root@master ~]# ctr -n k8s.io images push --plain-http=true --user admin:Harbor12345 10.24.2.200/library/maven:3.8.2-openjdk-8
[root@master ~]# ctr -n k8s.io images tag docker.io/library/cloud-ui:latest 10.24.2.200/library/cloud-ui:latest
[root@master ~]# ctr -n k8s.io images push --plain-http=true --user admin:Harbor12345 10.24.2.200/library/cloud-ui:latest
[root@master ~]#  ctr -n k8s.io images tag docker.io/library/openjdk:8-jdk-alpine 10.24.2.200/library/openjdk:8-jdk-alpine
[root@master ~]#  ctr -n k8s.io images push --plain-http=true --user admin:Harbor12345 10.24.2.200/library/openjdk:8-jdk-alpine
```

​		（2）部署Jenkins

为了便于管理，将Jenkins持续集成相关的所有资源对象都部署到jenkins命名空间下，新建命名空间：

```shell
[root@master ~]# kubectl create ns jenkins
```

部署Jenkins需要使用到一个拥有相关权限的Service Account，可以给该SA赋予一些必要的权限，也可以直接绑定一个cluster-admin的集群角色权限。定义一个名为jenkins-admin的SA，YAML文件如下：

```shell
[root@master ~]# vi jenkins-sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  labels:
    name: jenkins
┄ #三短横请手打
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jenkins-admin
  labels:
    name: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins-admin
    namespace: jenkins
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

创建SA对象：

```shell
[root@master ~]# kubectl -n jenkins apply -f jenkins-sa.yaml
```

使用Deployment部署Jenkins，使用动态存储StorageClass将容器的/var/jenkins_home目录持久化。YAML文件如下：

```shell
[root@master ~]# vi jenkins-deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  labels:
    app: jenkins
spec:
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      nodeName: master
      serviceAccountName: jenkins-admin
      containers:
      - name: jenkins
        image: jenkins/jenkins:latest
        imagePullPolicy: IfNotPresent
        securityContext: 
          runAsUser: 0
          privileged: true
        ports:
        - name: web
          containerPort: 8080
        - name: agent
          containerPort: 50000
        volumeMounts:
        - mountPath: /var/jenkins_home/
          name: jenkinshome
      volumes:
        - name: jenkinshome
          persistentVolumeClaim:
            claimName: jenkins-pvc
┄ #三短横请手打
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
spec:
  storageClassName: nfs-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

部署Jenkins：

```shell
[root@master ~]# kubectl -n jenkins apply -f jenkins-deploy.yaml
```

使用NodePort类型的Service来暴露Jenkins的8080端口，同时暴露50000端口，该端口主要用于Jenkins的Master和Slave之间的通信。YAML文件如下：

```shell
[root@master ~]# vi jenkins-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  labels:
    app: jenkins
spec:
  type: NodePort
  ports:
  - name: web
    port: 8080
    targetPort: web
    nodePort: 30880
  - name: agent
    port: 50000
    targetPort: agent
    nodePort: 30500
  selector:
    app: jenkins
```

创建Service对象：

```shell
[root@master ~]# kubectl -n jenkins apply -f jenkins-svc.yaml
```

查看Pod：

```shell
[root@master ~]# kubectl -n jenkins get pods
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-5948c7bbbc-rkrfd   1/1     Running   0          16s
```

查看Service：

```shell
[root@master ~]# kubectl -n jenkins get svc
NAME      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                          AGE
jenkins   NodePort   10.96.28.214   <none>        8080:30880/TCP,50000:30500/TCP   93s
```

通过NodePort访问Jenkins（http://NodeIP:NodePort），如图2所示。

![image002.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/058d83c886f0aa7d2d6180fbbdd14b23/image002.png)
图2 通过NodePort访问Jenkins

获取Jenkins密码：

```shell
[root@master ~]# kubectl -n jenkins exec deploy/jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword
988660fafc574960907c0e303918caff
```

输入密码后单击“继续”按钮，如图3所示。

![image003.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/2e44fd88f3e8d71133f5484cf7b074ce/image003.png)
图3 继续

将离线插件包拷贝到Jenkins：

```shell
[root@master ~]# kubectl -n Jenkins cp Jenkins-Slave/plugins/ jenkins-d99b47df4-6925b:/var/jenkins_home
```

重启Jenkins：

```shell
[root@master ~]# kubectl rollout restart deployment jenkins
deployment.apps/jenkins restarted
```

刷新Jenkins界面，选择“跳过插件安装”，创建一个用户jenkins，密码000000，如图4所示。

![image004.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/8d0bac507af6deaa28bf59904398ac5f/image004.png)
图4 创建管理员用户

单击“保存并完成”，如图5所示。

![image005.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/7f5edf4d305c69b0a0bc4d8fedef29e1/image005.png)
图5 实例配置

单击“保存并完成”，如图6所示。

![image006.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/e91d7303923e9defcd2017819c9986c0/image006.png)
图6 Jenkins安装完成

单击“开始使用Jenkins”按钮并使用新创建的用户登录Jenkins，如图7所示。

![image007.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/cc57f4ecc91b1bcd861bcdb6f8a58eb2/image007.png)
图7 登录Jenkins

（3）部署Jenkins Slave

持续构建与发布是日常工作中必不可少的一个步骤，目前大多公司都采用Jenkins集群来搭建符合需求的CI/CD流程，然而传统的Jenkins Slave一主多从方式会存在一些痛点，比如：

- 主Master发生单点故障时，整个流程都不可用；
- 每个Slave的配置环境不一样，差异化的配置导致管理起来非常不方便，维护起来也是比较费劲；
- 资源分配不均衡，有的Slave负载很高，而有的Slave处于空闲状态；
- 资源有浪费，每个Slave可能是裸机或者虚拟机，当Slave处于空闲状态时，也不会完全释放掉资源。

正因为上面的这些痛点，用户渴望一种更高效更可靠的方式来完成CI/CD流程，而容器技术能很好的解决这些问题，又特别是在Kubernetes集群环境下面能够更好来解决上面的问题，如图8所示是基于Kubernetes搭建Jenkins集群的简单示意图。

![image008.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/a9f7b4a1ab0eee5a9db5cba28a1d6b07/image008.png)
图8 基于Kubernetes搭建Jenkins集群

可以看到Jenkins Master和Jenkins Slave以Pod形式运行在Kubernetes集群的Node上，Master运行在其中一个节点，并且将其配置数据存储到一个Volume上去，Slave运行在各个节点上，并且它不是一直处于运行状态，它会按照需求动态的创建并自动删除。

这种方式的工作流程大致为：当Jenkins Master接受到Build请求时，会根据配置的Label动态创建一个运行在Pod中的Jenkins Slave并注册到Master上，当运行完Job后，这个Slave会被注销并且这个Pod也会自动删除，恢复到最初状态。

使用这种方式带来了哪些好处呢？

- 服务高可用，当Jenkins Master出现故障时，Kubernetes会自动创建一个新的Jenkins Master容器，并且将Volume分配给新创建的容器，保证数据不丢失，从而达到集群服务高可用。
- 动态伸缩，合理使用资源，每次运行Job时，会自动创建一个Jenkins Slave，Job完成后，Slave自动注销并删除容器，资源自动释放，而且Kubernetes会根据每个资源的使用情况，动态分配Slave到空闲的节点上创建，降低出现因某节点资源利用率高，还排队等待在该节点的情况。
- 扩展性好，当Kubernetes集群的资源严重不足而导致Job排队等待时，可以很容易的添加一个Kubernetes Node到集群中，从而实现扩展。

在Jenkins首页，选择“系统管理→节点和云管理”选项，如图9所示。

![image009.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/0fd5beaff4e416e8a1ceebf7491d0dad/image009.png)
图9 节点列表

依次点击“Clouds”、“New cloud”按钮，进入新建Cloud界面，输入Cloud名称，类型选择Kubernetes，如图10所示。

![image010.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/c78733908e27430592f3c90b067e158c/image010.png)
图10 配置云名称和类型

单击“Create”按钮完成创建，然后单击“Kubernetes Cloud details”按钮配置集群信息，首先配置Kubernetes地址，Kubernetes地址可以通过命令kubectl cluster-info获取，由于目前Jenkins运行在Kubernetes集群中，所以也可以使用Service的DNS形式的地址进行连接，地址为https://kubernetes.default.svc.cluster.local。命名空间选择jenkins，配置完成后单击“连接测试”按钮，如果返回“Connected to Kubernetes …”的提示信息就说明Jenkins已经可以和Kubernetes集群正常通信了，如图11所示。

![image011.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/434513e220b7bfed6dd561af2bfdcb53/image011.png)
图11 配置集群信息

然后配置Jenkins地址和Jenkins通道。Jenkins地址的格式为：http://Service名称.命名空间.svc.cluster.local:8080，因为Jenkins的服务名称为jenkins，且部署在jenkins命名空间下，所以Jenkins地址的值为http://jenkins.jenkins.svc.cluster.local:8080。Jenkins通道使用TCP协议和50000端口，格式为：Service名称.命名空间.svc.cluster.local:50000。配置完成后如图12所示。

![image012.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/19f2fe025dc7aaceaf0b1e0286cf5c6a/image012.png)
图12 配置Jenkins地址和Jenkins通道

单击最下方的“Pod Templates”按钮配置Jenkins Slave运行的Pod模板。Pod名称自定义，命名空间依旧使用jenkins，标签列表非常重要，流水线执行Job时需要用到该标签。配置完成后如图13所示。

![image013.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/6d9d404bb38dc0386ed8d6f309199377/image013.png)
图13 配置Jenkins Slave运行的Pod模板

然后配置容器模板，容器的名称必须是jnlp，这是默认拉起的容器，镜像使用jenkins/jnlp-slave:jdk11，该镜像在官方jnlp镜像的基础上集成了Docker、Kubectl 等一些实用的工具，另外需要将运行的命令和命令参数的值都删除掉，否则会失败。配置完成后如图14所示。

![image014.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/cb0c4752325c6acefd98dc2c638a863d/image014.png)
图14 配置容器模板

由于jnlp容器中默认使用的是Docker CLI，需要有Docker Daemon才能正常使用，通常情况下会采用Docker in Docker的方式将宿主机上的Docker Sock文件/var/run/docker.sock 挂载到容器中。但是此处Kubernetes集群使用的容器运行时是Containerd，没有安装Docker Daemon。所以这里以Pod的形式在集群中运行一个Docker Daemon服务，对应的YAML文件如下：

```shell
[root@master ~]# vi docker-dind.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    app: docker-dind
  name: docker-dind-data
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: nfs-storage
  resources:
    requests:
      storage: 5Gi
┄ #三短横请手打
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-dind
  labels:
    app: docker-dind
spec:
  selector:
    matchLabels:
      app: docker-dind
  template:
    metadata:
      labels:
        app: docker-dind
    spec:
      containers:
        - image: docker:dind
          imagePullPolicy: IfNotPresent
          name: docker-dind
          env:
            - name: DOCKER_HOST
              value: tcp://0.0.0.0:2375
            - name: DOCKER_TLS_CERTDIR # 禁用 TLS（最好别禁用）
              value: ""
          volumeMounts:
            - name: docker-dind-data-vol # 持久化docker根目录
              mountPath: /var/lib/docker/
          ports:
            - name: daemon-port
              containerPort: 2375
          securityContext:
            privileged: true # 需要设置成特权模式
      volumes:
        - name: docker-dind-data-vol
          persistentVolumeClaim:
            claimName: docker-dind-data
┄ #三短横请手打
apiVersion: v1
kind: Service
metadata:
  name: docker-dind
  labels:
    app: docker-dind
spec:
  ports:
    - port: 2375
      targetPort: 2375
  selector:
    app: docker-dind
```

创建上面的资源对象：

```shell
[root@master ~]# kubectl -n jenkins apply -f docker-dind.yaml
[root@master ~]# kubectl -n jenkins get pods -l app=docker-dind
NAME                           READY   STATUS    RESTARTS   AGE
docker-dind-6d7475cb4c-c6bkp   1/1     Running   0          109s
```

然后可以通过设置环境变量“DOCKER_HOST: tcp://docker-dind:2375”让Jenkins去连接Docker Dind服务，如图15所示。

![image015.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/e3dcf5ceba6ed3d8e6a31e510ef1e8b2/image015.png)
图15 连接Docker Dind服务

由于后续要编译的项目是一个Maven项目，所以需要用到mvn工具：

```shell
[root@master ~]# mkdir /home/jenkins
[root@master ~]# tar -zxf Jenkins-Slave/apache-maven-3.6.3-bin.tar.gz -C /home/jenkins/
[root@master ~]# mv /home/jenkins/apache-maven-3.6.3/ /home/jenkins/maven
[root@master ~]# cp -rf Jenkins-Slave/repository/ /home/jenkins/
```

将宿主机的/home/jenkins/maven和/home/jenkins/repository目录分别挂载到容器的/usr/local/maven和/home/jenkins/.m2/repository目录。还需要将目录/root/.kube挂载到容器的/root/.kube目录下面，如图16所示。这是为了能够在Pod的容器中能够使用Kubectl工具来访问Kubernetes集群，方便后面在Slave Pod部署Kubernetes应用。

![image016.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/2e26e0e424a99893a7a0295dd359d538/image016.png)
图16 挂载

此外还需要配置Jenkins Slave Pod的权限，配置Service Account，如图17所示。

![image017.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/3cd370d9e9138bb68bfca3ecb99bcfd2/image017.png)
图17 配置Service Account

Kubernetes插件的配置工作完成了，接下来添加一个Job任务，看是否能够在Slave Pod中执行，任务执行完成后看Pod是否会被销毁。

在Jenkins首页新建一个测试任务test，然后选择“构建一个自由风格的软件项目”类型，如图18所示。

![image018.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/b55567b06f9cc044519e133e3d00f08f/image018.png)
图18 新建一个测试任务

标签表达式处填入jnlp，必须和Slave Pod模板中的标签保持一致，如图19所示。

![image019.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/017c178d7f14e2864f19feb88581dca2/image019.png)
图19 标签表达式

在Build Steps区域选择“执行shell”，输入测试命令，如图20所示。

![image020.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/e58b5e4eda1852d6ad8cb17ba053e9d1/image020.png)
图20 输入测试命令

测试命令如下：

```shell
echo "测试 Kubernetes 动态生成 jenkins slave"
echo "==============docker in docker==========="
docker info
echo "=============kubectl============="
kubectl get pods
```

保存任务，并单击“立即构建”按钮构建项目。

查看Pod：

```shell
[root@master ~]# kubectl -n jenkins get pods
NAME                           READY   STATUS    RESTARTS       AGE
docker-dind-6d7475cb4c-c6bkp   1/1     Running   0              112m
jenkins-d99b47df4-6925b        1/1     Running   1 (121m ago)   134m
jnlp-40c55                     1/1     Running   0              6s
```

可以看到在点击立刻构建后生成了一个新Pod，这就是Jenkins Slave。任务执行完成后查看控制台输出，如图21所示。

![image021.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/8844a73acdb19a8e9dbc535f1aaf5323/image021.png)
图21 查看控制台输出

现在任务已经构建完成，此时再去集群查看Pod列表，Slave的Pod已经被删除了。

#### 2.2 部署 GitLab

（1）部署GitLab

编写GitLab资源清单文件：

```shell
[root@master ~]# vi gitlab-deploy.yaml
apiVersion: v1
kind: Service
metadata:
  name: gitlab
spec:
  type: NodePort
  ports:
  - port: 443
    nodePort: 30443
    targetPort: 443
    name: gitlab-443
  - port: 80
    nodePort: 30888
    targetPort: 80
    name: gitlab-80
  selector:
    app: gitlab
┄ #三短横请手打
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab
spec:
  selector:
    matchLabels:
      app: gitlab
  revisionHistoryLimit: 2
  template:
    metadata:
      labels:
        app: gitlab
    spec:
      containers:
      - image: gitlab/gitlab-ce:latest
        name: gitlab
        imagePullPolicy: IfNotPresent
        env:
        - name: GITLAB_ROOT_PASSWORD
          value: admin123
        - name: GITLAB_ROOT_EMAIL
          value: guorui@example.com
        - name: GITLAB_PORT
          value: "80"
        ports:
        - containerPort: 443
          name: gitlab-443
        - containerPort: 80
          name: gitlab-80
```

部署GitLab：

```shell
[root@master ~]# kubectl -n jenkins apply -f gitlab-deploy.yaml
```

查看Pod：

```shell
[root@master ~]# kubectl get pods
NAME                           READY   STATUS    RESTARTS       AGE
docker-dind-6d7475cb4c-c6bkp   1/1     Running   0              116m
gitlab-6f748c69dd-445wr        1/1     Running   0              39s
jenkins-d99b47df4-6925b        1/1     Running   1 (124m ago)   138m
```

查看GitLab Service：

```shell
[root@master ~]# kubectl -n jenkins get svc gitlab 
NAME     TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
gitlab   NodePort   10.96.82.242   <none>        443:30443/TCP,80:30888/TCP   59s
```

GitLab启动较慢，可以通过kubectl logs查看启动状态。启动完成后，通过NodePort访问GitLab（http://NodeIP:NodePort），如图22所示。

![image022.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/3d290d32945977f09a6f682d9dd8709d/image022.png)
图22 访问GitLab

登录GitLab，如图23所示。

![image023.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/8daeb20aa25efd0eb05095fc5aa832e2/image023.png)
图23 登录GitLab

（2）创建项目

单击“Create a project”，如图24所示。

![image024.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/eb0f49ae53d7e18db138cc26583f2896/image024.png)
图24 创建一个project

单击“Create blank project”按钮创建项目cloud-manager，可见等级选择“Public”，如图25所示。

![image025.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/3d455a494d33fa8367371d9e77048537/image025.png)
图25 创建项目cloud-manager

单击“Create project”，进入项目，如图26所示。

![image026.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/c509a21f965875a81859361cffdb2620/image026.png)
图26 进入项目

将源代码推送到项目中：

```shell
[root@master ~]# cd Jenkins-Slave/cloud-manager/
[root@master cloud-manager]# git config --global user.name "administrator"
[root@master cloud-manager]# git config --global user.email "admin@example.com"
[root@master cloud-manager]# git remote remove origin
[root@master cloud-manager]# git remote add origin http://10.24.2.200:30888/root/cloud-manager.git
[root@master cloud-manager]# git add .
[root@master cloud-manager]# git commit -m "initial commit"
[master (root-commit) 105c032] initial commit
[root@master cloud-manager]# git push -u origin master
Username for 'http://10.24.2.200:30888': root
Password for 'http://root@10.24.2.200:30888': 
Counting objects: 189, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (137/137), done.
Writing objects: 100% (189/189), 43.35 KiB | 0 bytes/s, done.
Total 189 (delta 40), reused 0 (delta 0)
remote: Resolving deltas: 100% (40/40), done.
To http://10.24.2.200:30888/root/cloud-manager.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

刷新网页，cloud-manager项目已经更新了，如图27所示。

![image027.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/29e870eefaca68cd4b31d5560049dbf7/image027.png)
图27 刷新项目

（3）配置GitLab

登录GitLab管理员界面（http:// NodePort:30888/admin），如图28所示。

![image028.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/c3b613de7c7b30752429525c67f26be3/image028.png)
图28 登录GitLab管理员界面

单击左侧导航栏的“Settings→Network”选项，设置“Outbound requests”，勾选“Allow requests to the local network from web hooks and services”，如图29所示。然后保存配置。

![image029.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/6380ec0cc85eb00271c1ab63741829fe/image029.png)
图29 设置

#### 2.3 部署RabbitMQ

部署RabbitMQ：

```shell
[root@master cloud-manager]# cd ../
[root@master Jenkins-Slave]# kubectl -n jenkins apply -f manifests/rabbit-deploy.yaml
```

查看Service：

```shell
[root@master Jenkins-Slave]# kubectl -n jenkins get svc rabbitmq
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP                                   PORT(S)                                        AGE
rabbitmq   NodePort   10.96.95.254   <none>   5672:5672/TCP,4369:27547/TCP,25672:27738/TCP   30s
```

访问Service（http://NodeIP:15672），使用用户名（guest/guest）密码登录，如图30所示。

![image030.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/8668f51ff2d9f880d58e2fd560ee8e22/image030.png)
图30 访问Service

#### 2.4 部署Nacos

Nacos致力于发现、配置和管理微服务。它提供了一组简单易用的特性集，帮助快速实现动态服务发现、服务配置、服务元数据及流量管理。使用Nacos可以更敏捷和容易地构建、交付和管理微服务平台。Nacos是构建以“服务”为中心的现代应用架构(例如微服务范式、云原生范式)的服务基础设施。

（1）部署Nacos

部署Nacos：

```shell
[root@master Jenkins-Slave]# kubectl -n jenkins apply -f manifests/mysql.yaml 
[root@master Jenkins-Slave]# kubectl -n jenkins apply -f manifests/nacos.yaml
```

查看Service：

```shell
[root@master Jenkins-Slave]# kubectl -n jenkins get svc
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                   AGE
docker-dind           ClusterIP   10.96.115.165   <none>        2375/TCP                                                  142m
gitlab                NodePort    10.96.82.242    <none>        443:30443/TCP,80:30888/TCP                                26m
jenkins               NodePort    10.96.28.214    <none>        8080:30880/TCP,50000:30500/TCP                            164m
mysql                 NodePort    10.96.199.46    <none>        3306:3306/TCP                                             26s
nacos-headless        NodePort    10.96.156.63    <none>        8848:8848/TCP,9848:9848/TCP,9849:9849/TCP,7848:7848/TCP   23s
rabbitmq              NodePort    10.96.95.254    <none>        5672:5672/TCP,4369:27547/TCP,25672:27738/TCP              7m54s
rabbitmq-management   NodePort    10.96.163.60    <none>        15672:15672/TCP                                           7m54s
```

访问Nacos（http://NodeIP:8848/nacos），如图31所示。

![image031.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/447c14a3a21129b4a5bf286e09661d97/image031.png)
图31 访问Nacos

输入用户名/密码（nacos/nacos），如图32所示。

![image032.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/982b6a09dc8b747d093c151177dbd708/image032.png)
图32 登录Nacos

单击“导入配置”按钮，将配置文件包nacos_config.zip导入系统，如图33所示。

![image033.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/e3d8657a121192e62d5f2bd93315b651/image033.png)

图33 导入配置

单击“编辑”按钮修改cloud-manage-public.yaml文件，将IP地址修改为实际IP地址，如图34所示。

![image034.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/6432ffe16f2b1a0992aba7955c469b41/image034.png)
图34 修改IP地址

修改完成后发布配置文件。

（2）导入数据

在选择启动Nacos的时候选择使用MySQL5.7，可以使用数据库连接工具直接连接该容器的MySQL服务（默认用户名：root，密码：root）导入cloud-manage-public的数据库。此处选择使用命令行导入数据。

查看MySQL Pod：

```shell
[root@master Jenkins-Slave]# kubectl -n jenkins get pods -l name=mysql
NAME          READY   STATUS    RESTARTS   AGE
mysql-v9tw4   1/1     Running   0          6m43s
```

导入数据库文件：

```shell
[root@master Jenkins-Slave]# kubectl -n jenkins cp cloud-manager/cloud_manage_public.sql mysql-v9tw4:/root
[root@master Jenkins-Slave]# kubectl -n jenkins exec -it mysql-v9tw4 -- bash
root@mysql-v9tw4:/# mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database cloud_manage_public;
Query OK, 1 row affected (0.01 sec)

mysql> use cloud_manage_public
Database changed
mysql>  source /root/cloud_manage_public.sql
Query OK, 0 rows affected (0.00 sec)
Query OK, 0 rows affected (0.00 sec)
Query OK, 0 rows affected, 1 warning (0.00 sec)
Query OK, 0 rows affected (0.02 sec)
Query OK, 0 rows affected (0.00 sec)

mysql> quit
Bye
```

#### 2.5 触发构建

Blue Ocean为开发人员提供了更具乐趣的Jenkins使用方式，它是从基础开始构建的，实现了一种全新的、现代风格的用户界面，有助于任何规模的团队实现持续交付。

（1）配置内部DNS解析

在Kubernetes集群中添加内部DNS解析：

```shell
[root@master Jenkins-Slave]# kubectl -n kube-system edit cm coredns
........
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
```

添加以下内容：

```shell
 hosts {
           10.24.2.200 nacos
           fallthrough
        }
```

重启CoreDNS以生效配置：

```shell
[root@master Jenkins-Slave]# kubectl -n kube-system rollout restart deploy coredns
```

（2）新建流水线任务

登录Jenkins，新建流水线任务cloud-manage-public，如图35所示。

![image035.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/84ffb3ff5317c519ea0889d0db28a7f2/image035.png)
图35 新建流水线任务

在定义域中选择“Pipeline script from SCM”，此选项指示Jenkins从源代码管理（SCM）仓库获取流水线。在SCM域中选择“Git”，然后输入“Repository URL”，如图36所示。

![image036.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/16aad51e78bab1a09e015bdbb014bafc/image036.png)
图36 配置流水线

在Credentials中选择“添加”按钮，凭据类型选择“Username with password”，然后输入对应信息，如图37所示。

![image037.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/0085883eb888edd822e2cdbbac4dcf31/image037.png)
图37 添加凭据

单击“保存”按钮，回到流水线中，在Credentials域选择刚才添加的凭证，如图38所示，最后保存流水线。

![image038.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/0977fddadd38c75b54ee76f8e853c80b/image038.png)

图38 选择凭证

（3）编写Jenkinsfile文件

要实现在Jenkins中的构建工作，可以有多种方式，这里采用比较常用的Pipeline方式。Pipeline，简单来说就是一套运行在Jenkins上的工作流框架，将原来独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂流程编排和可视化的工作。

Jenkins Pipeline有几个核心概念：

- Node：节点，一个Node就是一个Jenkins节点，Master或者Agent，是执行Step的具体运行环境；
- Stage：阶段，一个Pipeline可以划分为若干个Stage，每个Stage代表一组操作，比如：Build、Test、Deploy，Stage是一个逻辑分组的概念，可以跨多个Node；
- Step：步骤，Step是最基本的操作单元，由各类Jenkins插件提供，比如命令：sh ‘make’，就相当于在shell终端中执行make命令。

Pipeline有两种创建方法：一种是在Jenkins的Web UI界面中输入脚本，另一种是通过创建一个Jenkinsfile脚本文件放入项目源码库中。

一般推荐在Jenkins中直接从源代码控制(SCMD)中直接载入Jenkinsfile Pipeline这种方法。

登录GitLab进入项目cloud-manager，选择新建文件，如图39所示。

![image039.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/03a5189d042ac08b6bbfd2e948a999d9/image039.png)
图39 新建文件

将流水线脚本输入到Jenkinsfile中，如图40所示。

![image040.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/5dba7f8cf52f0236cacbd2f9beda66c3/image040.png)
图40 输入流水线脚本

完整的流水线脚本如下：

```shell
pipeline{
    agent none
    stages{
        stage('mvn-build'){
            agent { label 'jnlp'}
            steps{
                sh 'cd cloud-manage-common/ && /usr/local/maven/bin/mvn install'
                sh '/usr/local/maven/bin/mvn -B -DskipTests clean package'
                sh 'cd cloud-manage-public && /usr/local/maven/bin/mvn -B -DskipTests clean package'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
                sh 'sed -i "s|openjdk:8-jdk-alpine|10.24.2.200/library/openjdk:8-jdk-alpine|g" cloud-manage-public/Dockerfile'
                sh 'docker build -t 10.24.2.200/library/cloud-manage-public:latest -f cloud-manage-public/Dockerfile .'
                sh 'docker login 10.24.2.200 -u admin -p Harbor12345'
                sh 'docker push 10.24.2.200/library/cloud-manage-public:latest'
            }
        }
        stage('cloud-deploy'){
            agent { label 'jnlp'}
            steps{
                sh 'sed -i "s|cloud-manage-public:latest|10.24.2.200/library/cloud-manage-public:latest|g" manifests/cloud-deploy.yaml'
                sh 'sed -i "s|cloud-ui:latest|10.24.2.200/library/cloud-ui:latest|g" manifests/cloud-deploy.yaml'
                sh 'sed -i "s|http://IP:9998|http://10.24.2.200:9998|g" manifests/cloud-deploy.yaml'
                sh 'kubectl apply -f manifests/'
            }
        }
    }
}
```

该流水线脚本定义了两个阶段，第一个阶段mvn-build在Maven容器中构建代码，然后完成Docker镜像的构建和上传，第二个阶段cloud-deploy完成服务的自动发布。

（4）运行Blue Ocean

登录Jenkins首页，打开Blue Ocean，如图41所示。

![image041.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/8a79f19e9a17421ecdf295fbd0bcf18c/image041.png)

图41 打开Blue Ocean

选择cloud-manage-public项目，如图42所示。

![image042.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/660a62578b1a3affa37be3807949a584/image042.png)

图42 选择cloud-manage-public项目

单击“运行”按钮，如图43所示。

![image043.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/34ceea9124c25b3be299badf2a974ea5/image043.png)

图43 运行

单击正在构建的pipeline可以查看阶段视图，如图44所示。

![image044.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/aca2c2d7cbbbd2bba11085a0dd267461/image044.png)
图44 查看阶段视图

单击任意“>”符号可查看每个Step的构建详情，如图45所示。

![image045.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/9957735558259518033d4f9a1d7e95fb/image045.png)

图45 查看每个Step的构建详情

若Jenkins成功构建了Java应用，Blue Ocean界面会变为绿色。构建完成后如图46所示。

![image046.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/e9141850fd3ca492dd32f346eb6b5e28/image046.png)
图46 构建完成

退出阶段试图界面，如图47所示。可以看到，构建整个任务耗时2m12s。

![image047.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/2722f253e8c6794ec68587742be538e3/image047.png)
图47 退出阶段试图界面

（5）查看结果

登录Harbor镜像仓库，进入library项目，如图48所示。

![image048.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/540e106ae7e2863f97d8853d5cae1314/image048.png)
图48 进入library项目

可以看到项目中多了一个新构建的镜像cloud-manage-public，说明镜像构建并上传成功。

查看Pod：

```shell
[root@master Jenkins-Slave]# kubectl -n cloud-manage get pods
NAME                        READY   STATUS    RESTARTS   AGE
cloud-ui-767cc87cf9-6jqz8   1/1     Running   0          68s
my-cloud-c9c85c699-d6rnq    1/1     Running   0          68s
```

查看Service：

```shell
[root@master Jenkins-Slave]# kubectl -n cloud-manage get svc
NAME            TYPE       CLUSTER-IP        EXTERNAL-IP   PORT(S)          AGE
cloud-service   NodePort   192.99.254.167    <none>        9998:9998/TCP    78s
nginx-service   NodePort   192.110.117.230   <none>        8095:38095/TCP   78s
```

访问云管平台（http://NodeIP:38095），如图49所示。

![image049.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/5c953dbc5a63ab92d10f8f88d90c6a0c/image049.png)

图49 访问云管平台

登录云管平台（admin/000000），如图50所示。

![image050.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/878292fb9f7c4ef21efedd2fe0315d0e/image050.png)
图50 登录云管平台

至此，云管平台从构建及测试到交付的自动化CI/CD流程就实现了。通过该流程，系统就会通过自动构建应用并运行不同级别的自动化测试，可以减少部署新代码时所需的工作量，快速、轻松地将应用部署到生产环境中，实现了高效的持续集成/交付流程。