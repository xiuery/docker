## lnmp

- link
  * [mysql](https://hub.docker.com/_/mysql/)
  * [php-fpm](https://hub.docker.com/_/php/)
  * [nginx](https://hub.docker.com/_/nginx/)
  * [phpmyadmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
  * [phpmyadmin](https://docs.phpmyadmin.net/zh_CN/latest/index.html)
  * [docker hub](https://hub.docker.com/u/xiuery/)

- 设置WORKDIR
```
WORKDIR_LNMP=/home/projects/lnmp
```

- 删除lnmp
```
sudo docker ps -a | grep xiuery | awk '{print $1}' | xargs sudo docker stop
sudo docker ps -a | grep xiuery | awk '{print $1}' | xargs sudo docker rm
sudo rm -rf $WORKDIR_LNMP
sudo mkdir -p $WORKDIR_LNMP
sudo cd $WORKDIR_LNMP
```

- mysql
```
# 停止并删除已运行的
sudo docker ps -a | grep mysql | awk '{print $1}' | xargs sudo docker stop
sudo docker ps -a | grep mysql | awk '{print $1}' | xargs sudo docker rm
# 拉取mysql镜像
sudo docker pull xiuery/mysql:5.7.23
# 创建mysql数据目录
sudo rm -rf $WORKDIR_LNMP/mysql
sudo mkdir -p $WORKDIR_LNMP/mysql
# 启动mysql: -e 设置初始密码; -v 宿主机保存容器数据（非必须选项）
sudo docker run -d --name xiuery-mysql -p 3307:3306 \
  -v $WORKDIR_LNMP/mysql:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  xiuery/mysql:5.7.23
# 进入容器内部
sudo docker exec -ti xiuery-mysql /bin/bash
```

- php-fpm
```
# 停止并删除已运行的
sudo docker ps -a | grep php-fpm | awk '{print $1}' | xargs sudo docker stop
sudo docker ps -a | grep php-fpm | awk '{print $1}' | xargs sudo docker rm
# 拉取镜像
sudo docker pull xiuery/php-fpm:7.2.9 
# 官方镜像
sudo docker pull php:7.2-fpm
# 创建php目录
sudo rm -rf $WORKDIR_LNMP/php
sudo mkdir -p $WORKDIR_LNMP/php
# 启动php-fpm
sudo docker run -d --name xiuery-php-fpm -p 9000:9000 \
  -v $WORKDIR_LNMP/www:/var/www/html \
  xiuery/php-fpm:7.2.9 
# 启动php-fpm:内部连接mysql(这样可以在代码连接mysql时host=mysql)
# 注： 这种连接方式，从网页速度来看，比较慢（可能是忘了原因）
sudo docker run -d --name xiuery-php-fpm -p 9000:9000 \
  -v $WORKDIR_LNMP/www:/var/www/html \
  --link xiuery-mysql:mysql \
  xiuery/php-fpm:7.2.9 
# 进入容器内部
sudo docker exec -ti xiuery-php-fpm /bin/bash
# 容器内部安装php模块: pdo_mysql、mysqli
docker-php-ext-install MODULE
```

- php-fpm Dockerfile
```
FROM php:7.2-fpm
LABEL author=xiuery

FROM php:7.2-fpm
RUN docker-php-ext-install pdo_mysql \
	&& docker-php-ext-install mysqli \
	&& pecl install redis-4.0.1 \
	&& docker-php-ext-enable redis
RUN apt-get update && apt-get install -y \
		libfreetype6-dev \
		libjpeg62-turbo-dev \
		libpng-dev \
	&& docker-php-ext-install -j$(nproc) iconv \
	&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
	&& docker-php-ext-install -j$(nproc) gd
```

- nginx
```
# 停止并删除已运行的
sudo docker ps -a | grep nginx | awk '{print $1}' | xargs sudo docker stop
sudo docker ps -a | grep nginx | awk '{print $1}' | xargs sudo docker rm
# 拉取镜像
sudo docker pull nginx:1.14
# 创建www目录:启动nginx时将nginx的conf挂载在宿主机上
sudo mkdir -p $WORKDIR_LNMP/nginx/conf
sudo cp -r SOURCE $WORKDIR_LNMP/nginx
sudo mkdir -p $WORKDIR_LNMP/nginx/log
sudo rm -rf $WORKDIR_LNMP/www
sudo mkdir -p $WORKDIR_LNMP/www
# 启动nginx
sudo docker run -d --name xiuery-nginx \
  -p 9001:9001 \
  -p 9002:9002 \
  -p 9003:9003 \
  -v $WORKDIR_LNMP/nginx/conf:/etc/nginx \
  -v $WORKDIR_LNMP/nginx/log:/var/log/nginx \
  -v $WORKDIR_LNMP/www:/var/www/html \
  --link xiuery-php-fpm:php-fpm \
  nginx:1.14
# 进入容器内部
sudo docker exec -ti xiuery-nginx /bin/bash
```

- phpmyadmin:docker部署
```
# link到指定mysql容器
sudo docker run -d --name xiuery-mysql-admin -p 8999:80 \
  --link xiuery-mysql:db \
  phpmyadmin/phpmyadmin
# 指定使用哪台mysql server及port
sudo docker run -d --name xiuery-mysql-admin -p 8999:80 \
  -e PMA_HOST=192.168.199.115 \
  -e PMA_PORT=3307 \
  phpmyadmin/phpmyadmin
# 登录时选择mysql server
sudo docker run -d --name xiuery-mysql-admin -p 8999:80 \
  -e PMA_ARBITRARY=1 \
  phpmyadmin/phpmyadmin
```
- phpmyadmin:代码部署
```
# 启动lnmp服务
# 下载phpmyadmin源代码，修改配置文件
# cp config.sample.inc.php config.inc.php
# 修改配置中的$cfg: host、port...
```

