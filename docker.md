# Docker

* 隔离应用环境，避免冲突。确保应用运行环境一致性。
* 通过`dockerfile`定制`image`来实现持续集成、持续交付、部署。例如搭建和配置开发环境。
* 统一的管理服务。
* 开销比虚拟机小，可以做到秒级、甚至毫秒级的启动时间。

`docker run -ti -e "WEB_PORT=8080" --name [name] [repository:tag] [command]`

`docker run -d -p 80 --name static_web jamtur01/static_web nginx -g "daemon off;"`

参数|-
-|-
name|容器命名
rm|用完即扔的容器
i|开启STDIN
t|分配伪tty
d|守护式
p|`-p 80`容器80端口映射到宿主机随机分配端口<br>`-p 1080:80`容器80端口映射宿主机1080端口
P|公开在`Dockerfile`中通过`EXPOSE`指令公开的所有端口
v|卷映射，对宿主机目录的修改能及时反映到容器中<br>`-v /var/jenkins_home:/var/jenkins_home`宿主机目录：容器目录<br>`-v /data`<br>容器`/data`目录挂载到宿主机匿名目录，目的不是让在主机上修改，而是让多个容器共享<br>`--volumes-from [container_name]`<br>启动容器时，共享[container_name]中的volume。volume可通过`-v`或`Dockerfile`的`VOLUME`指定
net|指定新容器的运行网络
e|传递环境变量，只会在运行时有效
log-driver|`syslog`日志输出重定向
restart|`always`自启<br>`on-fialure`退出代码非0值自启
privileged|运行容器对Docker宿主机拥有root访问权限

命令|说明|-
-|-|-
docker start [name]|启动停止运行的容器
docker restart [name]|重新启动容器
docker attach [name]|附着到正在运行的容器
docker stop [name]|停止守护式容器
docker rm [name]|删除容器|`f`删除运行中的容器
docker ps|运行中的容器列表|`a`列出所有容器
docker port [name]|容器端口映射情况
docker inspect [name]|更多容器信息
docker logs [name]|容器日志|`f`等于`tail -f`<br>`tail`等于`tail -n`<br>`t`时间戳
docker top [name]|运行某个容器内的进程
docker stats [name]|统计信息
docker exec [name] [command]|容器内额外启动进程|`docker exec -ti [CONTAINER ID] /bin/bash`
docker images [repository]|列出可用镜像（所有或某仓库）
docker pull [repository:tag]|拉取镜像
docker history [repository]|探究构建过程
docker push [repository]|将镜像推送到Docker Hub
docker rmi [repository]|删除镜像
docker network create [networkname]|创建桥接网络
docker network inspect [networkname]|查看
docker network ls|列出当前系统中的所有网络
docker network rm|删除一个Docker网络
docker network connect app db2|添加已有容器到app网络
docker network disconnect app db2|从app网络中断开db2 容器

docker commit -m"" -a"" [id] [username/repository:tag] 提交镜像
m：提交信息
a：作者信息

## 基于Dockerfile构建新镜像

`mkdir static_web;cd static_web;vim Dockerfile`

```
# Version: 0.0.1
FROM ubuntu:14.04
MAINTAINER wangbin "wangbin199207@gmail.com"
# 替换镜像源
RUN sed -i 's/http:\/\/archive\.ubuntu\.com\/ubuntu\//http:\/\/mirrors\.163\.com\/ubuntu\//g' /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo 'Hi, I am in your container' > /usr/share/nginx/html/index.html
EXPOSE 80
```

```
docker build -t="jamtur01/static_web:v1" . 
docker build -t="jamtur01/static_web:v1" -f path/to/file
docker build -t="jamtur01/static_web:v1" git@github.com:jamtur01/docker-static_web
t：设置仓库和名称、标签
no-cache：略过缓存功能
.到本地目录找Dockerfile文件；
f非标准构建源的位置；
Git仓库
```

Dockerfile指令
```
RUN 镜像被构建时要运行的命令
CMD ["/bin/bash", "-l"] 容器启动时要运行的命令（类似docker run [command]，会覆盖CMD）
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"] run中指定的任何参数都会被当作参数再次传递给ENTRYPOINT
WORKDIR /opt/webapp/db 设置容器工作目录，entrypoint, cmd指定程序在此目录下执行
ENV RVM_PATH /home/rvm/ RVM_ARCHFLAGS="-arch i386" 构建过程设置环境变量
USER nginx 指定用户运行镜像
VOLUME ["/opt/project", "/data"] 向基于镜像创建的容器添加卷
ADD software.lic /opt/application/software.lic 将构建目录下的software.lic复制到镜像中
COPY conf.d/ /etc/apache2/ 相较于add，不会做提取和解压
LABEL version="1.0" location="New York" 为镜像添加元数据
STOPSIGNAL 停止容器时发送系统调用信号
ONBUILD ADD . /app/src 为镜像添加触发器
```

## 更换仓库、远程访问

低版本`vi /etc/docker/daemon.json`

  ```
  {
    "registry-mirrors": ["https://t5t8q6wn.mirror.aliyuncs.com"],//国内仓库
    "hosts":["unix:///var/run/docker.sock","tcp://0.0.0.0:2375"]//远程访问DOCKER_HOST=tcp://120.78.15.125:2375
    service docker restart
  }
  ```

新版19`vi /usr/lib/systemd/system/docker.service`

  `ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 --containerd=/run/containerd/containerd.sock --registry-mirror=https://t5t8q6wn.mirror.aliyuncs.com`


## Installing Jenkins with Docker

`docker pull jenkins/jenkins`

`docker run --restart=always -d -p 8080:8080 -v jenkins_home:/var/jenkins_home --name jenkins jenkins/jenkins:lts`

### 国内网络环境导致首次启动慢的问题

运行时添加环境变量使`jenkins`走代理更新插件

`--env JAVA_OPTS="-DsocksProxyHost=10.0.0.2 -DsocksProxyPort=8888"`

为了避免代理影响其他功能使用，下载完插件后可以把该容器删除，重新运行新的容器时把此配置删除。由于工作目录在宿主机所以插件启动时已更新完成。

### docker-in-docker实现方式

涉及到docker的命令映射到宿主机的命令执行`-v $(which docker):/usr/bin/docker`

涉及到docker通讯映射的宿主机执行`-v /var/run/docker.sock:/var/run/docker.sock`

`jenkins/jenkins`默认使用`jenkins`账户运行，需使用`sudo`命令执行`docker`命令，所以需要自定义`image`安装`sudo`命令并赋予`jenkins`账户权限。

Dockerfile:

  ```
  FROM jenkins/jenkins:lts
  USER root
  RUN apt-get -qqy update; apt-get install -qqy sudo
  RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
  USER jenkins
  ```

`docker build -t myjinkins .`

`docker run --restart=always -d -p 8080:8080 -v jenkins_home:/var/jenkins_home -v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock --name jenkins myjinkins`

## MySQL

`docker run --name mysql -e MYSQL_ROOT_PASSWORD=123465 -d -p 3306:3306 mysql:5.6.45`

## Docker Compose

命令|说明|-
-|-|-
`docker-compose up`|执行|`-d`守护进程方式运行
`docker-compose up redis`|启动某一个服务
`docker-compose start`|重启服务
`docker-compsoe stop`|
`docker-compose ps`|
`docker-compose logs`||`-f`
`docker-compose rm`|
