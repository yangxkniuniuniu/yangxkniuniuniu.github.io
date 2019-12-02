---
layout:     post
title:      大数据
subtitle:   大数据学习笔记
date:       2018-12-24
author:     owl city
header-img: img/post-bg-lamplight.jpg
catalog: true
tags:
    - Hadoop
    - 大数据
    - 分布式
---
## Hadoop
#### Hadoop安装与部署

**这里先介绍本地模式的安装与使用**

###### 本地模式安装
1. 配置Java
2. 官网下载hadoop的压缩包,然后在相应目录下解压缩: `tar -xzvf ...`
3. 修改配置文件
    - `hadoop-env.sh`里加上`JAVA_HOME`

    - `core-site.xml`

    ```html
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://127.0.0.1:50070</value>
        </property>
        <property>
            <name>io.file.buffer.size</name>
            <value>131072</value>
        </property>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>file:/Users/cody/temp</value>
        </property>
        <property>
            <name>hadoop.proxyuser.root.hosts</name>
            <value>*</value>
        </property>
        <property>
            <name>hadoop.proxyuser.root.groups</name>
            <value>*</value>
        </property>
</configuration>
    ```

    - `hdfs-site.xml`
    ```html
    <configuration>
    <!--    <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>localhost:9001</value>
        </property> -->
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/Users/cody/mydata/hadoop/dfs/name</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/Users/cody/mydata/hadoop/dfs/data</value>
        </property>
        <property>
             <name>dfs.replication</name>
             <value>1</value>
        </property>
        <property>
             <name>dfs.webhdfs.enabled</name>
             <value>true</value>
        </property>
        <property>
             <name>dfs.permissions</name>
             <value>false</value>
         </property>
         <property>
             <name>dfs.web.ugi</name>
             <value>supergroup</value>
         </property>
</configuration>
    ```
    - `mapred-site.xml`
    ```xml
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.address</name>
            <value>localhost:10020</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>localhost:19888</value>
        </property>
</configuration>
    ```

    - `yarn-site.xml`
    ```xml
    <configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>localhost:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>localhost:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>localhost:8031</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>localhost:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>localhost:8088</value>
        </property>
</configuration>

    ```
> 添加这几个配置就可以初步使用hadoop了

4. 配置完成后,格式化主节点的namenode,载入hadoop目录 `hadoop namenode -format` 或者`hdfs namenode -format`,出现successful说明成功,失败的话说明配置文件存在问题.

5. 执行`start-all.sh`,启动完成后使用`jps`查看任务是否正常启动,包含:`namenode`,`datanode`,`secondarynamenode`,`nodemanager`,`resourcemanager`

6. 如不全的话大部分原因都是配置文件的问题,检查一下或者google一下

7. 接下来可以使用webserver查看hadoop的应用和hdfs的情况
> 注意,在登录hdfs的web UI时,使用50070不行的话,可以使用9870,在3.0版本的hdfs访问端口发生了改变

#### Hadoop操作
######  HDFS操作
- 查看文件列表 `hadoop fs -ls /`
- 创建文件目录 `hadoop fs -mkdir /tmp`
- 删除文件 `hadoop fs -rm /tmp/data` , 删除目录 `hadoop fs -rmr /tmp`
- 上传文件 `hadoop fs -put ... /tmp/...`
- 下载文件 `hadoop fs -get /tmp/... ...`


## Kafka
#### 本地安装与部署
- 环境要求：java环境，brew工具
    - 1.brew install kafka
    - 2.启动kafka(kafka依赖zookeepee,需先启动zookeeper)：
        ```shell
        zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties
        ```

    - 3.创建topic：kafka-topic-create -zookeeper localhost:2181 -replication-factor 1 -partitions 1 -topic test
    - 4.发送消息： kafka-console-producer -broker-list localhost:9092 -topic test
    - 5.消费消息：kafka-console-consumer -bootstrap-server localhost:9092 -topic test -from-beginning

## Spark

## Flink
