---
title: docker基础知识
date: 2019-04-07 08:40:51
tags: 运维
id: 1554604484
---
# Docker
## 镜像
镜像是大家制作的程序包，里面可以只有 linux 环境，也可以包含特定的软件。
```sh
$ docker image ls # 列出本地所有镜像
$ docker image rm [imageName] # 删除制定镜像
$ docker image pull hello-world # 下载镜像到本地
```

## 容器
容器是运行中的镜像。docker 会把镜像复制一份，然后运行。
```sh
$ docker container run hello-world
```
hello-world 是 docker 官方的镜像。如果没有执行 image pull，会自动下载。

因为容器是镜像复制出来的，所以一旦运行后，也会有文件生成
```sh
$ docker container ls # 列出运行中的容器
$ docker container ls --all # 包括没在运行的容器
$ docker container rm [containerId] # 删除容器文件
```

## 开一个可交互的 nginx 容器
```sh
$ docker container run \
  -d \ # 后台运行
  -p 127.0.0.1:8080:80 \ # 容器内的 80 端口映射到本地8080端口
  -p 127.0.0.1:8081:443 \ # 容器内的 443 端口映射到本地8081端口
  --rm \ # 停止运行后删除容器
  --name mynginx \ #容器命名为mynginx
  --volume "$PWD/html":/usr/share/nginx/html \ #映射文件目录到本地
  --volume "$PWD/conf":/etc/nginx \ #映射配置文件到本地
  nginx
```

## 针对 linux
```sh
$ sudo groupadd docker # 添加 docker 组
$ sudo usermod -aG docker $USER # 将用户添加到 docker 组
$ sudo systemctl start docker # systemctl 开启服务
$ sudo service docker start # 或者 service 开启服务
```

# Dockerfile
下面来做一个 swoole 本地开发 dockerfile
```dockerfile
# 在 PHP 官方镜像基础上构建
FROM php:7.2-cli

# 复制本地文件到 /tmp
COPY swoole.tar.gz /tmp

# 指定接下来的工作目录
WORKDIR /app

RUN tar -xzf /tmp/swoole.tar.gz -C /tmp/swoole --strip-components=1 \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && docker-php-ext-enable \
  && rm -f /tmp/swoole.tar.gz \
  && rm -f /tmp/swoole \

# 映射文件
VOLUME ["/app","/www"]

# 暴露 8081 端口
EXPOSE 8081
```

编译运行
```sh
$ docker image build -t swoole-dev . # -t 后面跟镜像的名字
$ docker container run -p 8000:3000 swoole-dev
```

# Docker Compose
如果我们有多个服务，比如 wordpress + mysql，这个时候要先启动 mysql，然后根据 mysql 的 ip，改变 wordpress 的配置。这种情况，就要用到 docker compose 了。下面是一个例子
```yml
# 文件 docker-compose.yml
mysql:
    image: mysql:5.7
    environment:
        - MYSQL_ROOT_PASSWOD=123456
        - MYSQL_DATABASE=wordpress
web:
    image: wordpress
    links:
        - mysql
    environment:
        - WORDPRESS_DB_PASSWORD=123456
    ports:
        - "127.0.0.1:8080:80"
    working_dir: /var/www/html
    volumes:
        - wordpress: /var/www/html
```

执行
```sh
$ docker-compose up # 启动所有服务
$ docker-compose stop # 关闭所有服务
$ docker-compose rm # 删除容器
```

# 其他有用的命令
```sh
$ docker container start [containerId] #启动容器
$ docker container stop [containerId] #停止容器，先发出 SIGTERM 信号，然后发出 SIGKILL 信号
$ docker container kill [containerId] # 杀死运行中的容器，发出 SIGKILL 信号
$ docker container logs [containerId] # 查看 shell 输出
$ docker container exec -it [containerId] /bin/bash # 进入到容器内部 linux 环境
$ docker container cp [containerId]:[/path/to/file] # 复制容器
$ docker container inspect [containerId] # 查看容器 IP 地址
```

------------------------------
参考资料：  
http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html  
http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html  
http://www.ruanyifeng.com/blog/2018/02/nginx-docker.html