# Info

简介 : Docker是一个开源的应用容器Container引擎,可以让开发者将`应用及应用运行的环境`打包到一个轻量级 可移植的镜像中,然后发布到任何流行的Linux Windows等机器上;

`Docker 常用命令图`

![image-20231115204719696](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231115204719696.png)

# Mirror Image

简介 : 镜像是一种轻量级 、可执行的`独立软件包`,用来`打包软件运行环境和基于运行环境开发的软件`,它包含运行某个软件所需的所有内容,包括代码 、运行时、库、环境变量和配置文件;

```shell
# 从 docker hub 拉取镜像到本地
docker pull  
docker image pull
docker pull [OPTIONS] NAME[:TAG|@DIGEST]

# 将镜像推送到DockerRrgister
docker push
docker image push
docker push [OPTIONS] NAME[:TAG]

# 显示location镜像信息
docker images
docker image list
docker images [OPTIONS] [REPOSITORY[:TAG]]

# 在docker hub 上查询镜像,方便拉取
docker search
docker search [OPTIONS] TERM

# 删除本地一个或者多个镜像
docker rmi
docker image remove
docker rmi [OPTIONS] -f  IMAGE [IMAGE...]

# 将一个或者多个docker镜像保存本地压缩文件
docker save
docker image save
docker save [OPTIONS] -o  IMAGE [IMAGE...]

# 加载本地压缩文件到镜像
docker load
docker image load
docker load [OPTIONS] -i 
```

# Container

简介 :  Docker容器是Docker镜像的运行实例,容器提供了一种轻量级的虚拟化,可看作为一个虚拟机;

```shell
# 创建并启动容器
docker  run
docker  container run
docker run [OPTIONS] --name -d -p --network  -v  -e IMAGE [COMMAND] [ARG...]
# eg: 
# options -->  --name 自定义容器名字  -d 后台运行容器  --p 自定义端口号映射  --network 自定义网关
docker run  --name nginx  -d  -p 80:80  --network  hmall   mynginx:1.0

# 进入正在后台运行的容器
docker container exec 
docker exec [OPTIONS] -it  CONTAINER  COMMAND [ARG...]
#eg: 
# options --> -it 以伪终端的形式进入容器内,容器中启动一个新的bash shell会话;
docker exec  -it  test  /bin/bash

# 停止指定容器
docker stop
docker container stop


# 启动指定容器
docker start
docker container start

# 重新启动容器
docker restart
docker container restart

# 删除指定容器
docker rm
docker container remove
docker rm [OPTIONS] -f CONTAINER [CONTAINER...]

# 查看容器
docker ps 
docker container ps
docker ps [OPTIONS] -a

# 查看容器运行日志
docker logs
docker container logs
docker logs [OPTIONS] CONTAINER

# 查看容器详细信息
docker inspect
docker container inspect
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

# Volume

简介 : 数据卷是一个虚拟目录,是`容器内目录与宿主机目录之间映射的桥梁`;是一种数据持久化和共享数据的机制,它独立于容器声明周期的可管理的存储空间;

`/var/lib/docker/volumes`这个目录就是默认的存放所有容器数据卷的目录;

其下再根据数据卷名称创建新目录,格式为`/数据卷名/_data`;

![image-2023111719264 0736](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231117192640736.png) 

```shell
# 创建数据卷
docker volume create

# 查看所有数据卷
docker volume ls

# 删除指定的数据卷
docker volume rm

# 查看某个数据卷的详情
docker volume inspect

# 清楚数据卷
docker volume prune
```

*注意*：容器与数据卷的挂载要在创建容器时配置，对于创建好的容器，是不能设置数据卷的;

而且创建容器的过程中，数据卷会自动创建;

# Network

简介 : docker网络是docker中实现容器之间互相访问的机制,它是一个虚拟网络,可以通过桥接的方式组建;

```shell
# 创建网络
docker network create 
docker network create [OPTIONS] -d bridge  NETWORK

# 查看网络
docker network ls

# 查看某个网络的信息
docker network inspect 
docker network inspect [OPTIONS] NETWORK [NETWORK...]

# 将某个容器加入到某个网络;在同一个网络下的容器可以相互访问
docker network connect 
docker network connect [OPTIONS] NETWORK CONTAINER

# 删除网络
docker network rm
docker network rm NETWORK [NETWORK...]
```

# Dockerfile

简介 : DockerFile是一个文本文件,其中包含许多指令,通过指令构建出来一个新的镜像;

![image-20231117200845015](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231117200845015.png) 

```dockerfile
# eg : 搭建自己的java项目镜像

# 指定基础镜像
FROM openjdk:11.0-jre-buster
# 设置时区
ENV TZ=Asia/Shanghai
# 在镜像内部执行
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# 本地jar --> 拷贝到镜像内
COPY docker-demo.jar  /app.jar
# 运行时,执行指令为 java -jar /app.jar;不可被覆盖
ENTRYPOINT ["java", "-jar", "/app.jar"]

# 开始构建 文件名为 当前目录有 Dockerfile文件可以省略不写   . 表示当前目前
docker build -t  docker-demo:1.0    . 
```



```dockerfile
# eg : 搭建具有本地资源的nginx镜像
# 设置基础镜像
FROM nginx:latest
# 宿主机文件文件夹拷贝到镜像内
COPY  nginx  /app
# 替换镜像中部分文件
RUN cp  -r  /app/html/       /usr/share/nginx/
RUN cp  -r  /app/nginx.conf  /etc/nginx/nginx.conf
# 镜像默认端口号
EXPOSE 80
# 默认执行指令
ENTRYPOINT ["nginx-debug", "-g", "daemon off;"]
```

`推送镜像`

```shell
# 登录到 docker hub
docker login

# 标记镜像
docker tag mynginx:1.0  banne/mynginx:latest

# 推送镜像
docker push banne/mynginx:latest
```

![image-20231118103041666](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231118103041666.png) 



# DockerCompose

简介 : Docker Compose可以帮助我们实现互相关联的Docker容器的快速部署;它允许用户通过一个单独的docker-compose.yml模板文件来定义一组关联的应用容器;

```shell
# 从docker-compose.yaml文件构建并启动容器
docker compose up 

# 停止并删除通过docker-compose.yaml文件创建的容器 网络 卷
docker compose down

# 查看docker-compose.yaml文件创建的正在运行容器信息
docker compose ps

# 查看docker-compose.yaml文件创建容器的日志
docker compose logs
```

`docker-compose.yaml`

```yaml
# eg: docker-compose.yaml 部署 mysql 、hmall 

# 定义Docker Compose文件格式和版本
version: "3.8" 

# 定义创建和运行的服务
services:
  # 服务名称
  mysql:
  # 使用的镜像
    image: mysql      
    # 创建的容器名称   --> --name 
    container_name: mysql    
    # 映射的端口号  -->  -p
    ports:  
      - "3306:3306"
    # 设置环境变量 --> -e
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: 123
    # 挂载数据卷  --> -v  
    volumes:
      - "./mysql/conf:/etc/mysql/conf.d"
      - "./mysql/data:/var/lib/mysql"
    # 定义网络配置 --> --network 
    networks:
      - hm-net
 hmall: 
    # 指定 Dockerfile 文件夹的路径,使用Dockerfile创建镜像;
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: hmall
    ports:
      - "8080:8080"
    networks:
      - hm-net
    depends_on:
      - mysql      
      
# 创建网络  --> create network hmall    
networks:
  hm-net:
    name: hmall
    
    
docker run --name master-redis -dp 6379:6379 --network network-redis -v reids6379.conf:/usr/local/etc/redis/redis.conf  redis:7.2.3   redis-server /usr/local/etc/redis/redis.conf 
```

