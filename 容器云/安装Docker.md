## 安装Docker

```bash
#确定虚拟机能联网
ping baidu.com
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

#先安装yum工具包集合
yum install -y yum-utils                 

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

