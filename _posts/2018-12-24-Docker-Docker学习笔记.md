---
layout:     post
title:      Docker
subtitle:   Docker学习笔记
date:       2018-12-24
author:     owl city
header-img: img/post-bg-water.jpg
catalog: true
tags:
    - Helm
    - k8s
    - Docker
---


## Docker
- 在一个空文件夹下创建Dockerfile,app.py,requirement.txt,内容可从官网复制粘贴过来。

- `docker build -t friendlyhello` .  创建一个Docker image
	使用`docker image ls` 可以查看build的image

- `docker run -p 4000:80 friendlyhello`    运行app，然后访问localhost:4000。
	`docker run -d -p 4000:80 friendlyhello` 运行在后台
	`docker container ls` 列出当前正在运行的container，会有一个container ID
	`docker container stop ID`  ...来停止这个container

- 在repository中共享image：docker login进行登录，
	`docker tag image username/repository:tag `
`docker push username/repository:tag`

#### Service

- 在任意位置创建docker-compose.yml文件，将官网的内容复制进来，注意修改image的名字
	`https://docs.docker.com/get-started/part3/#your-first-docker-composeyml-file`
`docker swarm init`			
`docker stack deploy -c docker-compose.yml getstartedlab`    (在此处对app进行命名)
使用`docker service ls` 查看当前的service
`docker service ps getstartedlab_web` 或 `docker container ls -q` 查看具体的
然后就可以使用 `localhost:4000`运行

- 可以通过修改`docker-compose.yml`中的配置，然后重新启动`docker stack deploy`
`	docker stack deploy -c docker-compose.yml getstartedlab`

- 10.停止服务
	`docker stack rm getstartedlab`  		`docker swarm leave --force`

#### Swarm
- 本机创建swarm集群：`docker-machine create --driver virtualbox myvm1`
					`docker-machine create --driver virtualbox myvm2`
	然后启动node : `docker-machine start myvm1 // docker-machine start myvm2`
	notes:需要注意，得先下载好virtual-box
通过ssh启动myvm1，使其成为manager：`docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.99.100"`

- `docker-machine env myvm1` 配置shell能直接与myvm1进行交互
	按照输出，运行命令 `eval $(docker-machine env myvm1) `
	然后使用`docker-machine ls` 验证myvm1现在是一个激活的machine

- `docker stack deploy -c docker-compose.yml getstartedlab`  这样app就部署到了集群上

- 如果image在仓库中：使用`docker login`先登录，然后使用
	`docker stack deploy --with-registry-auth -c docker-compose.yml getstartedlab`

#### docker-compose
下载镜像`docker pull ...`
启动镜像 ` docker-composer up -d`
查看状态 ` docker ps -a`
查看日志  `docker logs containedID  `
扩展成集群 ` docker-compose scale kafka=2`
本地docker-compose启动kafka集群:

```yaml
version: '3'
services:
  zoo1:
    image: wurstmeister/zookeeper
    hostname: 192.168.121.174
    ports:
      - "2181:2181"
    container_name: zookeeper-2

  kafka1:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
      - "9996:9996"
      - "8200:8200"

   environment:
     KAFKA_ADVERTISED_HOST_NAME: 192.168.121.174
     KAFKA_ADVERTISED_PORT: 9092
     KAFKA_ZOOKEEPER_CONNECT: zoo1:2181
     KAFKA_BROKER_ID: 1
     KAFKA_OFFSET_TOPIC_REPLICATION_FACTOR: 1
     KAFKA_JMX_OPTS: "-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=127.0.0.1 -Dcom.sun.management.jmxremote.rmi.port=9996"
     JMX_PORT: 9996
     KAFKA_OPTS: "-javaagent:/test/jmx_prometheus_javaagent-0.3.1.jar=8200:/test/config.yaml"
   depends_on:
     - zoo1
   container_name: kafka-1

   volumes:
     - "/usr/local/software/jmx-export:/test"
     - "/var/run/docker.sock:/var/run/docker.sock"

 kafka2:
   image: wurstmeister/kafka
   ports:
     - "9093:9092"
     - "9997:9997"
     - "8201:8201"

   environment:
     KAFKA_ADVERTISED_HOST_NAME: 192.168.121.174
     KAFKA_ADVERTISED_PORT: 9093
     KAFKA_ZOOKEEPER_CONNECT: zoo1:2181
     KAFKA_BROKER_ID: 2
     KAFKA_OFFSET_TOPIC_REPLICATION_FACTOR: 1
     KAFKA_JMX_OPTS: "-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=127.0.0.1 -Dcom.sun.management.jmxremote.rmi.port=9997"
     JMX_PORT: 9997
     KAFKA_OPTS: "-javaagent:/test1/jmx_prometheus_javaagent-0.3.1.jar=8201:/test1/config.yaml"
   depends_on:
     - zoo1
   container_name: kafka-2

   volumes:
     - "/usr/local/software/jmx-export-2:/test1"
     - "/var/run/docker.sock:/var/run/docker.sock"

 grafana:
   image: grafana/grafana
   ports:
     - "3000:3000"

```


#### prometheus
启动:`./prometheus --config.file=.../prometheus.yml`
开启kafka-exporter
`docker run -ti --rm -p 9308:9308 danielqsj/kafka-exporter --kafka.server=192.168.121.174:9092 --kafka.server=192.168.121.174:9093 --kafka.server=192.168.121.174:9094`

### 使用Dokcerfile定制镜像
- FROM指定基础镜像
- 在有多行shell脚本时:

```shell
FROM debian:jessie

RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    ...
```

- 构造镜像
`docker build [选项] <上下文路径/URL/->`
这里的上下文路径会在创建镜像的时候将这个目录下的所有内容打包发送到docker引擎,这样Docker引擎收到这个上下文包后,展开就会获得构建镜像所需的一切文件.
在将文件复制进镜像时:
`COPY ./package.json /app/`的意思是将上下文路径下的这个json文件复制到镜像中
在有不想上传到Docker引擎的文件时,可以使用跟`.gitignore`一样的语法写一个`.dockerignore`
#### 其他`docker build`的用法
- 直接使用Git repo进行构建
`docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14`
- 用给定的tar包进行构建
`docker build http://server/context.tar.gz`

### Dockerfile指令详解
- COPY复制文件
格式 `COPY <源路径>  <目标路径>`

- ADD更高级的复制文件
格式和性质和COPY基本一致,再上面增加了一些功能
<源路径>可以是一个URL,Docker引擎会试图下载这个链接的文件到目标路径下去,下载后的文件权限自动设置为`600`

- CMD容器启动命令
CMD指令的格式和RUN相似:
    - shell格式:`CMD <命令>`
    - exec格式:`CMD ["可执行文件", "参数1", "参数2" ...]`
容器是进程,在启动容器的时候需要指定容器运行的程序和参数,CMD就是用来指定默认的容器的主进程的启动命令.
列:ubuntu镜像默认的CMD是`/bin/bash`,如果直接使用`docker run -it ubuntu`会直接进入bash,也可以在后面添加指令来完成其他动作.
***一般推荐用exec格式***
