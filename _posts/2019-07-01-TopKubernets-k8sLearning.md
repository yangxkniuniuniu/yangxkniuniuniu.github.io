---
layout:     post
title:      Kubernets
subtitle:   k8s学习笔记
date:       2019-07-01
author:     owl city
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - Helm
    - k8s
    - Docker
---

> - Create Date: 2019-12-03
> - Update Date: 2019-12-03

## Kubernetes
- [Kubernets中文文档](https://www.kubernetes.org.cn/k8s)

- 所有容器均在Pod中运行，一个Pod可以承载一个或者多个相关的容器

![Kubernetes设计架构](https://raw.githubusercontent.com/kubernetes/kubernetes/release-1.2/docs/design/architecture.png)

#### 核心组件
- `etcd` 保存整个集群的状态
- `apiserver` 提供了资源操作的唯一入口
- `controller manager` 负责维护集群的状态
- `scheduler` 负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上
- `kubelet` 负责维护容器的生命周期，同时也负责镜像、Volume(CVI)和网络(CNI)的管理
- `Container runtime` 负责镜像管理以及Pod和容器的真正运行(CRI）
- `kube-proxy` 负责为Service提供cluster内部的服务发现和负载均衡
#### 其他组件
- `kube-dns` 负责为整个集群提供DNS服务
- `Ingress Controller` 为服务提供外网入口
- `Heapster` 提供资源监控
- `Dashboard` 提供GUI
- `Federation` 提供跨可用区的集群
- `Fluentd-elasticsearch` 提供集群日志采集，存储与查询

#### 分层架构
- 核心层
对外提供API构建高层的应用，对内提供插件式应用执行环境
- 应用层
部署和路由
- 管理层
系统度量，自动化以及策略管理
- 接口层
kubectl命令行工具、客户端SDK以及集群联邦
- 生态系统
 - Kubernetes外部：日志、监控、配置管理、CI、CD、Workflow等
 - Kubernetes内部：CRI、CNI、CVI、镜像仓库等


## Helm
- [Helm文档](https://docs.helm.sh/)

- [Helm中文文档](https://whmzsu.github.io/helm-doc-zh-cn/quickstart/quickstart-zh_cn.html)

#### Helm基础
- 基础命令
    - 创建默认的Chart : `helm create mychart`

    - 自定义service.yaml文件时,通过`helm lint .`来检查yaml文件是否正确


- Helm作为Kubernetes的包管理：
    - 创建新的chart
    - chart打包成tgz
    - 上传chart到chart仓库或者从仓库下载chart
    - 在Kubernetes集群中安装或者卸载chart
    - 管理Helm安装的chart的发布周期

- Helm的三个重要概念：
    - chart:包含了创建Kubernetes的一个应用实例的必要信息
    - config:包含了应用发布配置信息
    - release:是一个chart及其配置的一个运行实例

- Helm组件
    - Helm Client
    - Tiller server

- chart chart是描述相关的一组Kuberbetes资源的文件集合，通过创建为特定目录树的文件，将它们打包到版本化的压缩包，然后进行部署。

#### Helm结构：

```yaml
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  requirements.yaml   # OPTIONAL: A YAML file listing dependencies for the chart
  values.yaml         # The default configuration values for this chart
  charts/             # A directory containing any charts upon which this chart depends.
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```


#### Templates
Templates目录下是yaml文件的模板

查看helm是否安装成功`kubectl -n kube-system get pods|grep tiller`

- 模板Templates和值Values
当helm渲染charts时，会传递templates目录中的每个文件
提供values的两种方法：
- `values.yaml`  该文件可以包含默认值
- `helm install -f`来指定一个包含值得yaml文件

在values.yaml文件提供的值可以从.Values模板中的对象访问，也可以在模板中访问其他预定义的数据片段。
一下是预定义的，可用于每个模板，并且不能被覆盖，区分大小写
- `Release.Name`    :   release的名称

- `Release.Time`    :   chart版本上次更新的时间

- `Release.Namespace`   :   chart ralease 发布的namespace

- `Release.Service` :   处理release的服务，通常是Tiller

- `Release.IsUpgrade`   :   如果当前神操作是升级或者回滚，则设置为true

- `Release.IsInstall`   :   如果当前操作是安装，则true

- `Chart`   :   `Chart.yaml`的内容

使用`helm install --values=myvalues.yaml kafka-ops` 这个yaml文件会被合并到默认values文件中,如果value.yaml中存在相同key则覆盖

#### helm
- `helm install –-dry-run –-debug ./` 验证chart模板和配置

#### kubectl
- 根据yaml配置文件一次性创建service,rc
`kubectl create -f my-service.yaml`
- 查看资源对象
    * 查看所有pod列表 `kubectl get pod `
    * 查看RC和service列表 `kubectl get rc,svc`
- 描述资源对象
    * 显示Node详细信息 `kubectl describe node`
    * 显示pod详细信息 `kubectl describe pod`
- 删除资源对象
    * 给予pod.yaml定义的名称删除pod `kubectl delete -f pod.yaml`
    * 删除所有包含某个label的pod和service `kubectl delete pod,svc -l ...`
    * 删除所有Pod   `kubectl delete pod --all`
- 执行容器的命令
    * 执行pod的date命令 `kubectl exec <pod-name> -- date`
    * 通过bash获得pod中某个容器中的TTY，相当于登录容器
    `kubectl exec -it <pod-name> -c <container-name> -- bash`
- 查看容器的日志
    `kubectl logs <pod-name>`
- 查看状态
    `kubectl describe svc garish-squid-0-external`
- 在外部访问
**将本地工作站上的 5557 端口转发到 redis-master pod 的 5556 端口**
`kubectl port-forward <podname> 5557:5556`

***当Helm安装和升级charts时，charts中的kubernetes对象及所有依赖项:***
- 聚合成一个单一的集合
- 按类型排序，然后按名称排序
- 按该顺序创建/更新


#### StatefulSet
-  一个Headless Service,名称为ngnix，用来控制网络
-  StatefulSet,名称为web,用Spec来指明有3个ngnix容器被启动在单独的Pod中。
-  volumeClaimTemplates通过使用PersistentVolumes提供stable storge
