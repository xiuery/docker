## docker常用命令

- 镜像
```
# 查看本地所有镜像
sudo docker images
# 搜索镜像
sudo docker search IMAGE
# 下载镜像:默认latest
sudo docker pull IMAGE
# 下载指定TAG的镜像
sudo docker pull IMAGE:TAG
# 删除镜像
sudo docker rmi IMAGE [IMAGE...]
``` 

- 容器
```
# 启动新容器(IMAGE:TAG)
sudo docker run -t -i --name x_centos centos:latest /bin/bash
# 启动新容器(不开放端口)：一般作为父容器使用，比如DB
sudo docker run -d --name x_web training/webapp python app.py
# 启动新容器(开放默认端口)
sudo docker run -d -P --name x_web training/webapp python app.py
# 启动新容器(开放一个指定端口)
sudo docker run -d -p 8080:5000 --name x_web training/webapp python app.py
# 启动新容器(开放多个指定端口)
sudo docker run -d -p 8080:5000 -p 8081:5000 --name x_web training/webapp python app.py
# 交互式启动新容器:进入容器内部操作
sudo docker run -t -i --name x_web training/webapp /bin/bash
# 查看所有容器：根据status判断容器状态
sudo docker ps -a
# 查看最近启动的容器
sudo docker ps -l
# 查看容器底层信息：返回一个json
sudo docker inspect CONTAINER
# 查看容器底层信息：按传入的键过滤
sudo docker inspect -f '{{ .NetworkSettings.IPAddress }}' CONTAINER
# 查看容器绑定的端口
sudo docker port CONTAINER
# 查看容器标准日志输出
sudo docker logs -f CONTAINER
# 查看容器进程
sudo docker top CONTAINER
# 停止容器
sudo docker stop CONTAINER [CONTAINER...]
# 启动停止的容器
sudo docker start CONTAINER [CONTAINER...]
# 删除容器
sudo docker rm CONTAINER [CONTAINER...]
# 停止所有容器
sudo docker stop $(sudo docker ps -q)
# 删除所有容器
sudo docker rm $(sudo docker ps -aq)
# 停止并删除所有容器
sudo docker stop $(sudo docker ps -q) & sudo docker rm $(sudo docker ps -aq)
```

- 自定义镜像
```
# 1.存在的镜像更新并提交
sudo docker run -t -i centos /bin/bash
# 在镜像内修改，然后exit
# 提交镜像：0b2616b0e5a8为进入镜像时的id
sudo docker commit -m="add something" -a="xiuery" 0b2616b0e5a8 xiuery/centos:v2
# 推送至docker hub
sudo docker login
sudo docker push xiuery/centos
# 2.Dockerfile指令
# Dockerfile:
FROM ubuntu:16.04
MAINTAINER xiuery <xiuery@xiuery.com>
RUN apt-get -y update
RUN apt-get -y install json
# 运行命令
sudo docker build -t="xiuery/ubuntu:16.04-v2" .
```

- 管理容器数据
```
# 无数据卷
sudo docker run -t -i --name x_centos centos:latest /bin/bash
# 添加一个数据卷
sudo docker run -t -i --name x_centos -v /home/projects/test centos:latest /bin/bash
# 挂载一个主机目录作为卷(可读写)
sudo docker run -t -i --name x_centos -v /home/projects/test:/home/projects/test centos:latest /bin/bash
# 挂载一个主机目录作为卷(只读)
sudo docker run -t -i --name x_centos -v /home/projects/test:/home/projects/test:ro centos:latest /bin/bash

```












