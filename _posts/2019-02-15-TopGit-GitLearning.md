---
layout:     post
title:      Git
subtitle:   Git学习
date:       2019-02-15
author:     owl city
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - Git
    - CI
    - Docker
---

> - Create Date: 2019-12-03
> - Update Date: 2019-12-03

## Git基础
#### 基础命令
- `git init`,在本地创建版本库的时候使用
- `git clone`,从服务端将项目的版本库克隆下来
- `git fetch`,从远程代码库更新数据到本地代码库。它只是将代码更新到本地代码库，需要check out或与当前工作分支merge才能在工作目录中看到代码的改变。
- `git pull`,从远程代码库更新数据到本地代码库，并与当前工作分支合并，等同于Fetch+Merge
- `git push`,将本地代码库中已经commit的数据推送到指定的remote，没有commit的数据，不会push

#### 名词解释
- `HEAD`,指向正在工作的本地分支的指针
- `Master分支`,主分支，所有提供给用户使用的正式版本，都在这个主分支上发布
- `Tags`,用来记录重要的版本历史
- `Origin`,默认的remote的名称

#### 工作流程
* 对代码进行修改
* 完成某项功能后commit,可以重复进行，知道觉得可以推送到服务器上时，往下执行
* pull
* 如果存在冲突，解决冲突
* Push,将数据提交到服务器的代码库

#### 将项目上传到gitlab中
- `git config --global user.name <用户名>`
- `git config --global user.email <邮箱>`
- `git init`
- `git remote add origin <项目链接>`
- `git pull --rebase origin master`
**如果git中的代码出现更新,需要将最新版的代码pull到本地库中**
- `git add .`
- `git commit -m ""`
- `git push -u origin master`

#### 分支
- 查看分支:
    - `git branch -a` 查看所有分支
    - `git branch` 查看本地分支
- 切换分支:
    - `git checkout dev` 切换到本地dev分支
- 保存和恢复进度:
    - 在`git checkout`时,如当前有未提交代码,且扔想切换分支,可以使用`git stash`进行暂存, `git stash save "messages..."`可以添加一些注释
    - `git stash pop` 恢复之前的进度
    - `git stash list` 显示保存进度的列表
- 创建分支:
    - `git branch dev` 创建本地分支
    - `git push origin dev` 提交本地分支到远程分支
    - `git checkout master; git checkout -b dev` 从master分支拉取新分支dev
- 删除分支
    - `git branch -d dev` 删除本地分支
    - 删除远程分支:
    ```shell
    git branch -r -d origin/dev
    git push origin :dev
    ```
- 合并分支:
切换到master分支 `git checkout master`
对develop分支进行合并`git merge --no-ff dev`
- 合并分支时遇到冲突
git已经进行了合并,需要我们手动处理冲突,再提交没有冲突的文件

## GitLab CI/CD
![GitLab CI/CD]( https://docs.gitlab.com/ee/ci/img/cicd_pipeline_infograph.png)

#### gitLab-ci配置与使用
[详细配置文档链接](https://docs.gitlab.com/ee/ci/yaml/README.html)
- 简单示例1
```yaml
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
- 简单示例2
启动一个job,在centos系统下安装一个kafka节点，然后开启kafka-exporter服务。
```yaml
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

#### 组件介绍
- jobs
    - `script` （required）可以直接执行系统命令，任务由Runners接管并且由服务器中的runner执行

    - `image` 使用docker镜像

    - `services` 使用docker服务

    - `stage` (默认test)

    - `type`   stage的别名

    - `variables`  定义在job层的变量

    - `only` 定义job将会运行的tags和分支名字

    - `except` 定义job将不会运行的tags和分支名字
    >`only`和`except`可以使用下列关键字:               `branches`,`tags`,`api`,`external`,`pipelines`,`pushes`,`schedules`,`triggers`,`web`

    - `tags` 定义一个tags的列表，来用作选Runner，使用这个得保证Runner里面已经定义了这些。

    - `allow_failure` 允许job fail,失败的job不会导致提交状态失败,true or false

    - `when` 定义什么时候去运行job,可以使`on_success`,`on_failure`,`always`,`manual`

    - `dependencies` 定义一个job依赖的其他jobs，可以传递artifacts

    - `artifacts` 定义**job artifacts**列表，在job运行成功以后能够在GitLab UI中能够下载的文件夹目录

    - `cache` 定义一个需要被缓存的文件列表

    - `before_script`

    - `after_script`

    - `environment` 定义job的部署环境

    - `coverage` 为job定义代码覆盖设置

    - `retry` 失败后重试

- pages
    pages是一个独特的job，它能够上传静态内容到GitLab上来服务网页。它有独特的语法，有两个必须指定:
    - 1.static content必须在public/文件夹下
    - 2.artifacts 路径必须被制动到public/目录下
    ```yaml
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
- images 用于docker镜像

- services 用于docker镜像

- before_script和after_script

- stages stages中的元素顺序决定了对应job的执行顺序：build , test , deploy 【默认也是这个顺序

- stage stage定义每一个job依赖的stages。相同的stage会并行执行

- variables
    添加变量，并在job环境中起作用
    要想在一个指定的job中关闭全局变量，可以定义一个空的hash：
    ```yaml
    job_name:
      variables: {}
    ```
- cache 指定需要在job之间缓存的文件或者目录，只能使用该项目工作空间内的路径

- key 定义缓存的作用域
