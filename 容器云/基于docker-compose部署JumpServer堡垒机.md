[TOC]

## 基于docker-compose部署JumpServer堡垒机



Compose 使用的三个步骤：

首先，Dockerfile 定义应用程序的环境。

然后，使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。

最后，执行 docker-compose up 命令来启动并运行整个应用程序。

涉及到的工具与技术包括：

- MySQL：数据库管理系统。
- Redis：完全开源，遵守 BSD 协议，是一个高性能的 key-value 数据库。
- Docker、Dockerfile：容器引擎，所有应用最终都要以Docker容器运行，Dockerfile是Docker镜像定义文件。
- Koko：为SSH Server和Web Terminal Server 。用户可以使用自己的账户通过SSH或者Web Terminal访问 SSH协议和Telnet协议资产。
- Nginx：高性能的HTTP和反向代理web服务器。
- Guacamole：一个无客户端远程桌面。
- Core：JumpServer管理后台。



#### 2.1 基础环境准备

将软件包JumpServer.tar.gz下载到本地。

```shell
[root@master ~]# curl -O http://mirrors.douxuedu.com/competition/JumpServer.tar.gz
```

解压软件包：

```shell
[root@master ~]# tar -zxf JumpServer.tar.gz
```

导入镜像：

```shell
[root@master ~]# docker load -i JumpServer/centos_7.9.2009.tar
```

#### **2.2** 配置环境变量

编写环境变量脚本：

```shell
[root@master ~]# cd JumpServer/
[root@master JumpServer]# vi .env
Version=v2.5.3

# MySQL
DB_HOST=mysql
DB_PORT=3306
DB_USER=root
DB_PASSWORD=000000
DB_NAME=root

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=8URXPL2x3HZMi7xoGTdk3Upj

# Core
SECRET_KEY=B3f2w8P2PfxIAS7s4URrD9YmSbtqX4vXdPUL217kL9Xtetetss
BOOTSTRAP_TOKEN=7Q11Vz6R2J6BLAdO

LOG_LEVEL=ERROR
```

#### 2.3 容器化部署MySQL

（1）编写Dockerfile

编写yum源：

```shell
[root@master JumpServer]# vi local.repo 
[jumpserver]
name=jumpserver
baseurl=file:///opt/jumpserverrepo
enabled=1
gpgcheck=0
```

编写数据库初始化脚本：

```shell
[root@master JumpServer]# vi mysql_init.sh 
#!/bin/bash
#

function config_mysql {
    if [ $DB_PORT != 3306 ]; then
        if [ ! "$(cat /etc/my.cnf | grep port=)" ]; then
            sed -i "10i port=$DB_PORT" /etc/my.cnf
        else
            sed -i "s/port=.*/port=$DB_PORT/g" /etc/my.cnf
        fi
    fi
}

if [ ! -d "/var/lib/mysql/$DB_NAME" ]; then
    config_mysql
    mysqld --initialize-insecure --user=mysql --datadir=/var/lib/mysql
    mysqld --daemonize --user=mysql
    sleep 5s
    mysql -uroot -e "create database $DB_NAME default charset 'utf8' collate 'utf8_bin';grant all on $DB_NAME.* to '$DB_USER'@'%' identified by '$DB_PASSWORD';flush privileges;";
    mysql --version
    tail -f /var/log/mysqld.log
else
    config_mysql
    mysqld --daemonize --user=mysql
    mysql --version
    tail -f /var/log/mysqld.log
fi
```

编写Dockerfile文件：

```shell
[root@master JumpServer]# vi Dockerfile-mysql 
FROM centos:7.9.2009
MAINTAINER Chinaskills
WORKDIR /opt
ARG Version=v2.5.3
ENV Version=${Version} \
    LANG=en_US.utf8

ADD jumpserverrepo.tar.gz .
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/
RUN set -ex \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "LANG=en_US.utf8" > /etc/locale.conf \
    && yum install -y mysql-community-server \
    && yum clean all \
    && rm -rf /var/cache/yum/*

COPY mysql_init.sh .
RUN chmod 755 ./mysql_init.sh

CMD ["./mysql_init.sh"]
```

（2）构建镜像

```shell
[root@master JumpServer]# docker build -t jms_mysql:v1.0 -f Dockerfile-mysql .
```

#### 2.4 容器化部署Redis

（1）编写Dockerfile

编写Redis初始化脚本：

```shell
[root@master JumpServer]# vi redis_init.sh 
#!/bin/bash
#

function config_redis {
    if [ $REDIS_PORT != 6379 ]; then
        sed -i "s/port 6379/port $REDIS_PORT/g" /etc/redis.conf
    fi
    if [ ! "$(cat /etc/redis.conf | grep -v ^\# | grep requirepass | awk '{print $2}')" ]; then
        sed -i "481i requirepass $REDIS_PASSWORD" /etc/redis.conf
    else
        sed -i "s/requirepass .*/requirepass $REDIS_PASSWORD/" /etc/redis.conf
    fi
}

config_redis
redis-server /etc/redis.conf
```

编写Dockerfile文件：

```shell
[root@master JumpServer]# vi Dockerfile-redis 
FROM centos:7.9.2009
MAINTAINER Chinaskills
WORKDIR /opt
ARG Version=v2.5.3
ENV Version=${Version} \
    LANG=en_US.utf8
ADD jumpserverrepo.tar.gz .
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/
RUN set -ex \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "LANG=en_US.utf8" > /etc/locale.conf \
    && echo "net.core.somaxconn = 1024" >> /etc/sysctl.conf \
    && echo "vm.overcommit_memory = 1" >> /etc/sysctl.conf \
    && yum install -y redis \
    && sed -i "s/protected-mode yes/protected-mode no/g" /etc/redis.conf \
    && sed -i "s/bind 127.0.0.1/bind 0.0.0.0/g" /etc/redis.conf \
    && sed -i "561i maxmemory-policy allkeys-lru" /etc/redis.conf \
    && yum clean all \
    && rm -rf /var/cache/yum/*

COPY redis_init.sh .
RUN chmod 755 ./redis_init.sh
CMD ["./redis_init.sh"]
```

（2）构建镜像

```shell
[root@master JumpServer]# docker build -t jms_redis:v1.0 -f Dockerfile-redis .
```

#### 2.5 容器化部署Nginx

（1）编写Dockerfile

编写Dockerfile文件：

```shell
[root@master JumpServer]# vi Dockerfile-nginx 
FROM centos:7.9.2009
MAINTAINER Chinaskills
WORKDIR /opt
ARG Version=v2.5.3
ENV Version=${Version} \
    LANG=en_US.utf8

ADD jumpserverrepo.tar.gz .
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/
ADD nginx/lina-v2.5.3.tar.gz .
ADD nginx/luna-v2.5.3.tar.gz .
RUN set -ex \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "LANG=en_US.utf8" > /etc/locale.conf \
    && yum install -y nginx \
    && echo > /etc/nginx/conf.d/default.conf \
    && mv luna-${Version} luna \
    && mv lina-${Version} lina \
    && rm -rf /opt/*.tar.gz \
    && yum clean all \
    && rm -rf /var/tmp/yum*
COPY nginx/nginx.conf /etc/nginx/

CMD ["nginx", "-g", "daemon off;"]
```

（2）构建镜像

```
[root@master JumpServer]# docker build -t jms_nginx:v1.0 -f Dockerfile-nginx .
```

#### 2.6 容器化部署Koko

（1）编写Dockerfile

编写Koko初始化脚本：

```shell
[root@master JumpServer]# vi koko_init.sh 
#!/bin/bash
#

sleep 5s
while [ "$(curl -I -m 10 -L -k -o /dev/null -s -w %{http_code} ${CORE_HOST}/api/health/)" != "200" ]
do
    echo "wait for jms_core ready"
    sleep 2
done

if [ ! $LOG_LEVEL ]; then
    export LOG_LEVEL=ERROR
fi

cd /opt/koko
./koko
```

编写Dockerfile文件：

```shell
[root@master JumpServer]# vi Dockerfile-koko 
FROM centos:7.9.2009
MAINTAINER Chinaskills
WORKDIR /opt
ARG Version=v2.5.3
ENV Version=${Version} \
    LANG=en_US.utf8

ADD koko/kubectl.tar.gz .
ADD koko/koko-v2.5.3-linux-amd64.tar.gz .
RUN mkdir /opt/kubectl-aliases/
ADD koko/kubectl_aliases.tar.gz /opt/kubectl-aliases/
ADD jumpserverrepo.tar.gz .
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/

RUN set -ex \
   && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
   && echo "LANG=en_US.utf8" > /etc/locale.conf \
   && yum install -y mysql-community-client bash-completion \
   && mv koko-${Version}-linux-amd64 koko \
   && chown -R root:root koko \
   && mv /opt/koko/kubectl /usr/local/bin/ \
   && chmod 755 ./kubectl \
   && chown root:root ./kubectl \
   && mv kubectl /usr/local/bin/rawkubectl \
   && chown -R root:root /opt/kubectl-aliases/ \
   && chmod 755 /opt/koko/init-kubectl.sh \
   && rm -rf /opt/*.tar.gz \
   && yum clean all \
   && rm -rf /var/cache/yum*

COPY koko_init.sh .
RUN chmod 755 ./koko_init.sh

CMD [ "./koko_init.sh" ]
```

（2）构建镜像

```shell
[root@master JumpServer]# docker build -t jms_koko:v1.0 -f Dockerfile-koko .
```

#### 2.7 容器化部署Guacamole

（1）编写Dockerfile

编写Guacamole初始化脚本：

```shell
[root@master JumpServer]# vi guacamole_init.sh 
#!/bin/bash
#

if [ ! $JUMPSERVER_KEY_DIR ]; then
    export JUMPSERVER_KEY_DIR=/config/guacamole/data/keys
fi
if [ ! $GUACAMOLE_HOME ]; then
    export GUACAMOLE_HOME=/config/guacamole
fi
if [ ! $GUACAMOLE_LOG_LEVEL ]; then
    export GUACAMOLE_LOG_LEVEL=ERROR
fi
if [ ! $JUMPSERVER_ENABLE_DRIVE ]; then
    export JUMPSERVER_ENABLE_DRIVE=true
fi
if [ ! $JUMPSERVER_RECORD_PATH ]; then
    export JUMPSERVER_RECORD_PATH=/config/guacamole/data/record
fi
if [ ! $JUMPSERVER_DRIVE_PATH ]; then
    export JUMPSERVER_DRIVE_PATH=/config/guacamole/data/drive
fi
if [ ! $JUMPSERVER_CLEAR_DRIVE_SESSION ]; then
    export JUMPSERVER_CLEAR_DRIVE_SESSION=true
fi
if [ ! $JUMPSERVER_CLEAR_DRIVE_SCHEDULE ]; then
    export JUMPSERVER_CLEAR_DRIVE_SCHEDULE=24
fi

rm -rf /config/tomcat9/logs/*

sleep 5s
while [ "$(curl -I -m 10 -L -k -o /dev/null -s -w %{http_code} ${JUMPSERVER_SERVER}/api/health/)" != "200" ]
do
    echo "wait for jms_core ready"
    sleep 2
done

/etc/init.d/guacd start
cd /config/tomcat9/bin && ./startup.sh

echo "Guacamole version $Version, more see https://www.jumpserver.org"
echo "Quit the server with CONTROL-C."

if [ ! -f "/config/guacamole/data/log/info.log" ]; then
    echo "" > /config/guacamole/data/log/info.log
fi

tail -f /config/guacamole/data/log/info.log
```

编写Dockerfile文件：

```shell
[root@master JumpServer]# vi Dockerfile-guacamole 
FROM centos:7.9.2009
WORKDIR /opt
MAINTAINER Chinaskills
ARG Version=v2.5.3
ENV Version=${Version} \
    LANG=en_US.utf8

ADD guacamole/apache-tomcat-7.0.33.tar.gz /config
COPY guacamole/ssh-forward.tar.gz /config
COPY guacamole/guacamole-client-v2.5.3.tar.gz /config
COPY guacamole/guacamole-server-1.2.0.tar.gz /config
COPY guacamole/docker-guacamole-v2.5.3.tar.gz /config
ADD jumpserverrepo.tar.gz .
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/
RUN set -ex \
    && yum clean all \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "LANG=en_US.utf8" > /etc/locale.conf \
    && yum install -y make gcc java-1.8.0-openjdk \
    && yum install -y cairo-devel libjpeg-turbo-devel libpng-devel libtool uuid-devel \
    && yum install -y ffmpeg-devel freerdp-devel pango-devel libssh2-devel libtelnet-devel libvncserver-devel libwebsockets-devel pulseaudio-libs-devel openssl-devel libvorbis-devel libwebp-devel \
    && mkdir -p /config/guacamole/lib /config/guacamole/extensions /config/guacamole/data/log/ /config/guacamole/data/record /config/guacamole/data/drive \
    && cd /config \
    && mv apache-tomcat-7.0.33 tomcat9 \
    && rm -rf tomcat9/webapps/* \
    && sed -i 's/# export/export/g' /root/.bashrc \
    && sed -i 's/# alias l/alias l/g' /root/.bashrc \
    && echo "java.util.logging.ConsoleHandler.encoding = UTF-8" >> /config/tomcat9/conf/logging.properties \
    && mkdir /config/docker-guacamole \
    && tar -xf docker-guacamole-${Version}.tar.gz -C /config/docker-guacamole --strip-components 1 \
    && rm -rf docker-guacamole-${Version}.tar.gz \
    && chown -R root:root /config/docker-guacamole \
    && tar -xf guacamole-server-1.2.0.tar.gz -C /config/docker-guacamole \
    && cd /config/docker-guacamole \
    && cd guacamole-server-1.2.0 \
    && ./configure --with-init-dir=/etc/init.d \
    && make \
    && make install \
    && ldconfig \
    && cd /config \
    && tar -xf ssh-forward.tar.gz -C /bin/ \
    && chmod 755 /bin/ssh-forward \
    && tar -xf guacamole-client-${Version}.tar.gz \
    && cp guacamole-client-${Version}/guacamole-*.war /config/tomcat9/webapps/ROOT.war \
    && cp guacamole-client-${Version}/guacamole-*.jar /config/guacamole/extensions/ \
    && cd /config \
    && mv /config/docker-guacamole/guacamole.properties /config/guacamole/ \
    && yum -y remove libwinpr \
    && rm -rf /config/docker-guacamole \
    && yum clean all \
    && rm -rf /var/tmp/yum*

COPY guacamole_init.sh .
RUN chmod 755 ./guacamole_init.sh

CMD ["./guacamole_init.sh"]
```

（2）构建镜像

```shell
[root@master JumpServer]# docker build -t jms_guacamole:v1.0 -f Dockerfile-guacamole .
```

#### 2.8 容器化部署Core

（1）编写Dockerfile

编写Core初始化脚本：

```shell
[root@master JumpServer]# vi core_init.sh 
#!/bin/bash
#

sleep 5s
while ! nc -z $DB_HOST $DB_PORT;
do
    echo "wait for jms_mysql ready"
    sleep 2s
done

while ! nc -z $REDIS_HOST $REDIS_PORT;
do
    echo "wait for jms_redis ready"
    sleep 2s
done

if [ ! -f "jumpserver/config.yml" ]; then
    echo > jumpserver/config.yml
fi

if [ ! $LOG_LEVEL ]; then
    export LOG_LEVEL=ERROR
fi

if [ ! $WINDOWS_SKIP_ALL_MANUAL_PASSWORD ]; then
    export WINDOWS_SKIP_ALL_MANUAL_PASSWORD=True
fi

source /opt/py3/bin/activate
cd /opt/jumpserver && ./jms start
```

编写Dockerfile文件：

```shell
[root@master JumpServer]# vi Dockerfile-core 
FROM centos:7.9.2009
MAINTAINER Chinaskills
ARG Version=v2.5.3
ENV Version=${Version} \
    LANG=en_US.utf8
WORKDIR /opt
ADD core/packages.tar.gz .
ADD jumpserverrepo.tar.gz .
RUN rm -rf /etc/yum.repos.d/*
COPY local.repo /etc/yum.repos.d/
ADD core/jumpserver-v2.5.3.tar.gz .
RUN set -ex \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "LANG=en_US.utf8" > /etc/locale.conf \
    && yum install -y gcc nc \
    && yum install -y python36 python36-devel \
    && mv jumpserver-${Version} jumpserver \
    && chown -R root:root jumpserver \
    && yum install -y $(cat /opt/jumpserver/requirements/rpm_requirements.txt) \
    && python3.6 -m venv /opt/py3 \
    && source /opt/py3/bin/activate \
    && pip3 install --no-index --find-links=/opt/packages/ -r /opt/jumpserver/requirements/requirements.txt \
    && yum clean all \
    && rm -rf /var/cache/yum/* \
    && rm -rf /opt/*.tar.gz \
    && rm -rf /var/cache/yum* \
    && rm -rf ~/.cache/pip

COPY core_init.sh .
RUN chmod 755 ./core_init.sh

CMD ["./core_init.sh"]
```

（2）构建镜像

```shell
[root@master JumpServer]# docker build -t jms_core:v1.0 -f Dockerfile-core .
```

#### 2.9 编排部署JumpServer

（1）编写docker-compose.yml编排部署文件：

```
[root@master JumpServer]# vi docker-compose.yaml
version: '3'
services:
  mysql:
    image: jms_mysql:v1.0
    container_name: jms_mysql
    restart: always
    tty: true
    environment:
      DB_PORT: $DB_PORT
      DB_USER: $DB_USER
      DB_PASSWORD: $DB_PASSWORD
      DB_NAME: $DB_NAME
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - jumpserver

  redis:
    image: jms_redis:v1.0
    container_name: jms_redis
    restart: always
    tty: true
    environment:
      REDIS_PORT: $REDIS_PORT
      REDIS_PASSWORD: $REDIS_PASSWORD
    volumes:
      - redis-data:/var/lib/redis/
    networks:
      - jumpserver

  core:
    image: jms_core:v1.0
    container_name: jms_core
    restart: always
    tty: true
    environment:
      SECRET_KEY: $SECRET_KEY
      BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN
      LOG_LEVEL: $LOG_LEVEL
      DB_HOST: $DB_HOST
      DB_PORT: $DB_PORT
      DB_USER: $DB_USER
      DB_PASSWORD: $DB_PASSWORD
      DB_NAME: $DB_NAME
      REDIS_HOST: $REDIS_HOST
      REDIS_PORT: $REDIS_PORT
      REDIS_PASSWORD: $REDIS_PASSWORD
    depends_on:
      - mysql
      - redis
    volumes:
      - core-data:/opt/jumpserver/data
    networks:
      - jumpserver

  koko:
    image: jms_koko:v1.0
    container_name: jms_koko
    restart: always
    privileged: true
    tty: true
    environment:
      CORE_HOST: http://core:8080
      BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN
      LOG_LEVEL: $LOG_LEVEL
    depends_on:
      - core
      - mysql
      - redis
    volumes:
      - koko-data:/opt/koko/data
    ports:
      - 2222:2222
    networks:
      - jumpserver

  guacamole:
    image: jms_guacamole:v1.0
    container_name: jms_guacamole
    restart: always
    tty: true
    environment:
      JUMPSERVER_SERVER: http://core:8080
      BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN
      GUACAMOLE_LOG_LEVEL: $LOG_LEVEL
    depends_on:
      - core
      - mysql
      - redis
    volumes:
      - guacamole-data:/config/guacamole/data
    networks:
      - jumpserver

  nginx:
    image: jms_nginx:v1.0
    container_name: jms_nginx
    restart: always
    tty: true
    depends_on:
      - core
      - koko
      - mysql
      - redis
    volumes:
      - core-data:/opt/jumpserver/data
    ports:
      - 81:80
    networks:
      - jumpserver

volumes:
  mysql-data:
  redis-data:
  core-data:
  koko-data:
  guacamole-data:

networks:
  jumpserver:
```

（2）服务部署

```shell
[root@master JumpServer]# docker-compose up -d
Creating network "jumpserver_jumpserver" with the default driver
Creating volume "jumpserver_mysql-data" with default driver
Creating volume "jumpserver_redis-data" with default driver
Creating volume "jumpserver_core-data" with default driver
Creating volume "jumpserver_koko-data" with default driver
Creating volume "jumpserver_guacamole-data" with default driver
Creating jms_mysql ... done
Creating jms_redis ... done
Creating jms_core  ... done
Creating jms_guacamole ... done
Creating jms_koko      ... done
Creating jms_nginx     ... done 
```

（3）查看镜像列表：

```shell
[root@master JumpServer]# docker images
REPOSITORY      TAG              IMAGE ID       CREATED              SIZE
jms_core        v1.0             4c51761b3049   About a minute ago   1.7GB
jms_guacamole   v1.0             7c2546cfd4f2   8 minutes ago        1.2GB
jms_koko        v1.0             51df43356d9f   14 minutes ago       948MB
jms_nginx       v1.0             e727f0013704   15 minutes ago       710MB
jms_redis       v1.0             079c0f97ed5d   16 minutes ago       658MB
jms_mysql       v1.0             18b3c12ca2b6   17 minutes ago       1.61GB
centos          7.9.2009         eeb6ee3f44bd   24 months ago        204MB
```

（4）查看服务：

```shell
[root@master JumpServer]# docker-compose ps
    Name              Command          State                    Ports                  
---------------------------------------------------------------------------------------
jms_core        ./core_init.sh         Up                                              
jms_guacamole   ./guacamole_init.sh    Up                                              
jms_koko        ./koko_init.sh         Up      0.0.0.0:2222->2222/tcp,:::2222->2222/tcp
jms_mysql       ./mysql_init.sh        Up                                              
jms_nginx       nginx -g daemon off;   Up      0.0.0.0:81->80/tcp,:::81->80/tcp        
jms_redis       ./redis_init.sh        Up      
```

（5）在浏览器上通过http://NodeIP:81端口访问服务，如图1所示：

![图1.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/b9439137d819875a9b7efd263ffbd2a8/%E5%9B%BE1.png)
图1 访问服务

登录（admin/admin）堡垒机，如图2所示：

![图2.png](https://ydy-resources-prod.obs.cn-north-4.myhuaweicloud.com/resource/2e573fd96997dcff19df987db61134b5/%E5%9B%BE2.png)

图2 登录堡垒机