## docker

- link
[docker hub](https://hub.docker.com/u/xiuery/)
[docker document](https://docs.docker.com/get-started/)

- INSTALL
```
# uninstall old versions
sudo apt-get remove docker docker-engine docker.io
# install using the repository
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# install
sudo apt-get update
# install the latest version
sudo apt-get install docker-ce
# or install special version
apt-cache madison docker-ce
sudo apt-get install docker-ce=<VERSION>
```

- UNINSTALL
```
sudo apt-get purge docker-ce
sudo rm -rf /var/lib/docker
```

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
# build失败的镜像
sudo docker images | grep none
sudo docker images | grep none | awk '{print $3}' | xargs sudo docker rmi [-f]
# 删除搜索到的镜像
sudo docker images -a | grep xiuery | awk '{print $3}' | xargs sudo docker rmi [-f]
``` 

- 容器
```
# 启动新容器(IMAGE:TAG)
sudo docker run -t -i --name CONTAINER-NAME centos:latest /bin/bash
# 启动新容器(不开放端口)：一般作为父容器使用，比如DB
sudo docker run -d --name CONTAINER-NAME training/webapp python app.py
# 启动新容器(开放默认端口)
sudo docker run -d -P --name CONTAINER-NAME training/webapp python app.py
# 启动新容器(开放一个指定端口)
sudo docker run -d -p 8080:5000 --name CONTAINER-NAME training/webapp python app.py
# 启动新容器(开放多个指定端口)
sudo docker run -d -p 8080:5000 -p 8081:5000 --name CONTAINER-NAME training/webapp python app.py
# 交互式启动新容器:进入容器内部操作
sudo docker run -t -i --name CONTAINER-NAME training/webapp /bin/bash
# 进入后台运行的容器内部
sudo docker exec -ti CONTAINER-NAME /bin/bash
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
# 重启容器
sudo docker restart CONTAINER [CONTAINER...]
# 删除容器
sudo docker rm CONTAINER [CONTAINER...]
# 重启所有容器
sudo docker restart $(sudo docker ps -q)
# 停止所有容器
sudo docker stop $(sudo docker ps -q)
# 删除所有容器
sudo docker rm $(sudo docker ps -aq)
# 停止并删除所有容器
sudo docker stop $(sudo docker ps -q) & sudo docker rm $(sudo docker ps -aq)
# 搜索关键字容器停止并删除
sudo docker ps -a | grep xiuery | awk '{print $1}' | xargs sudo docker stop
sudo docker ps -a | grep xiuery | awk '{print $1}' | xargs sudo docker rm
```

- 自定义镜像：存在的镜像修改后提交
```
sudo docker run -t -i centos /bin/bash
# 在镜像内修改，然后exit
# 提交镜像：0b2616b0e5a8为进入镜像时的id
sudo docker commit -m="add something" -a="xiuery" 0b2616b0e5a8 xiuery/centos:v2
# 推送至docker hub
sudo docker login
sudo docker push xiuery/centos
```
- Dockerfile指令
```
# FROM指令用于指定其后构建新镜像所使用的基础镜像
FROM <IMAGE>:<TAG> 
# LABEL用于为镜像添加元数据，元数以键值对的形式指定
# 指定后可以通过docker inspect image查看
LABEL <key>=<value> [<key>=<value> ...]
# MAINTAINER用于指定镜像作者
MAINTAINER XIUERY
# RUN用于在镜像容器中执行命令
RUN <command> 
RUN ["executable", "param1", "param2"]
# CMD用于指定在容器启动时所要执行的命令：
# 仅可指定一次，指定多次时，会覆盖前的指令
# docker run命令也会覆盖CMD命令
CMD ["executable","param1","param2"]
CMD ["param1","param2"]
CMD command param1 param2
# ENTRYPOINT用于给容器配置一个可执行程序：
# 仅可指定一次，指定多次时，会覆盖前的指令
# 可以通过docker run --entrypoint重写ENTRYPOINT
# docker run命令中指定的任何参数，都会被当做参数再次传递给ENTRYPOINT
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
# EXPOSE用于指定容器在运行时监听的端口: 配合docker run -p PORT:PORT使用
EXPOSE <port> [<port>...]
# ENV用于设置环境变量
ENV <key> <value> [<key> <value> ...]
ENV <key>=<value> [<key>=<value> ...]
# ADD用于复制构建环境中的文件或目录到镜像中
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
# COPY同样用于复制构建环境中的文件或目录到镜像中
COPY <src>... <dest>
COPY ["<src>",... "<dest>"] 
# VOLUME用于创建挂载点，即向基于所构建镜像创始的容器添加卷
VOLUME ["/data"] 
# USER用于指定运行镜像所使用的用户
USER daemon
# WORKDIR用于在容器内设置一个工作目录
# 通过WORKDIR设置工作目录后，RUN、CMD、ENTRYPOINT、ADD、COPY等指令都会在该目录下执行
# 使用docker run运行容器时，可以通过-w参数覆盖构建时所设置的工作目录
WORKDIR /path/to/workdir 
# ARG用于指定传递给构建运行时的变量
ARG <name>[=<default value>]
# ONBUILD用于设置镜像触发器
ONBUILD [INSTRUCTION] 
# STOPSIGNAL用于设置停止容器所要发送的系统调用信号
# 所使用的信号必须是内核系统调用表中的合法的值，如：9、SIGKILL
STOPSIGNAL signal
# SHELL用于设置执行命令（shell式）所使用的的默认shell类型
SHELL ["executable", "parameters"]

# 构建镜像
sudo docker build -t="IMAGE:TAG" .
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













