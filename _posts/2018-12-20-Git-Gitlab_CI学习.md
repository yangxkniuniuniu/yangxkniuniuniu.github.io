---
layout:     post
title:      Gitlab CI学习
subtitle:   Gitlab CI
date:       2018-12-19
author:     owl city
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - Git
    - CI
    - Docker
---

## GitLab CI/CD
![GitLab CI/CD]( https://docs.gitlab.com/ee/ci/img/cicd_pipeline_infograph.png)

### Pull下来centos的docker镜像，并能够进入container使用centos镜像
`docker pull centos:5.11`
`docker run --rm -it centos:7 bash`
### gitLab使用之ssh和depolydocker run --rm -it centos:7 bash keys
[具体连接]（http://ruikye.com/2016/10/19/github-ssh-keys/）

### 使用步骤(clone项目以及新建README.md然后更新和上传)
`sudo git clone git@git.saybot.net:cody.yang/mytest.git`
`cd mytest`
`sudo touch README.md`
`sudo git add README.md`
`sudo git commit -m "add README"`
`sudo git push -u origin master`


## gitLab-ci配置文档
[详细配置文档链接](https://docs.gitlab.com/ee/ci/yaml/README.html)
#### 简单示例1
```
stages:
  - runtest
image: centos:7.0.1406
job1:
  stage: runtest
  script:
    - echo 1
  tags:
    - docker
```
#### 简单示例2
启动一个job,在centos系统下安装一个kafka节点，然后开启kafka-exporter服务。
```
test1:
  image:
     name: centos:latest
  services:
    - name: spotify/kafka
      alias: kafka
  script:
    - yum install -y nc
    - yum install -y wget
    - wget https://github.com/danielqsj/kafka_exporter/releases/download/v1.2.0/kafka_exporter-1.2.0.linux-amd64.tar.gz
    - tar -xzvf kafka_exporter-1.2.0.linux-amd64.tar.gz
    - cd kafka_exporter-1.2.0.linux-amd64
    - nohup ./kafka_exporter --kafka.server=kafka:9092 &
    - nc -zv kafka 9092
    - nc -zv localhost 9308
    - curl -o metrics-details http://localhost:9308/metrics
    - cat metrics-details
```
### jobs
- script （required）
可以直接执行系统命令，任务由Runners接管并且由服务器中的runner执行
- image
使用docker镜像
- services
使用docker服务
- stage (默认test)
- type   stage的别名
- variables  定义在job层的变量
- only
定义job将会运行的tags和分支名字
- except
定义job将不会运行的tags和分支名字
`only`和`except`可以使用下列关键字:
`branches`,`tags`,`api`,`external`,`pipelines`,`pushes`,`schedules`,`triggers`,`web`
- tags
定义一个tags的列表，来用作选Runner，使用这个得保证Runner里面已经定义了这些。
- allow_failure
允许job fail,失败的job不会导致提交状态失败,true or false
- when
定义什么时候去运行job,可以使`on_success`,`on_failure`,`always`,`manual`
- dependencies
定义一个job依赖的其他jobs，可以传递artifacts
- artifacts
定义**job artifacts**列表，在job运行成功以后能够在GitLab UI中能够下载的文件夹目录
- cache
定义一个需要被缓存的文件列表
- before_script
- after_script
- environment
定义job的部署环境
- coverage
为job定义代码覆盖设置
- retry
失败后重试

## pages
pages是一个独特的job，它能够上传静态内容到GitLab上来服务网页。它有独特的语法，有两个必须指定:
1.static content必须在public/文件夹下
2.artifacts 路径必须被制动到public/目录下
```
pages:
  stage: deploy
  script:
    - mkdir .public
    - cp -r * .public
    - mv .public public
  artifacts:
    paths:
      - public
  only:
    - master
```
## images
用于docker镜像
## services
用于docker镜像
### before_script和after_script
## stages
stages中的元素顺序决定了对应job的执行顺序：build , test , deploy 【默认也是这个顺序
## stage
stage定义每一个job依赖的stages。相同的stage会并行执行
#### variables
添加变量，并在job环境中起作用
要想在一个指定的job中关闭全局变量，可以定义一个空的hash：
```
job_name:
  variables: {}
```
#### cache
指定需要在job之间缓存的文件或者目录，只能使用该项目工作空间内的路径
- key 定义缓存的作用域
