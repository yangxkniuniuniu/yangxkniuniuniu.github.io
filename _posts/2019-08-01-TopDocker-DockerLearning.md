---
layout:     post
title:      Docker
subtitle:   Docker学习笔记
date:       2019-08-01
author:     owl city
header-img: img/post-bg-water.jpg
catalog: true
tags:
    - Helm
    - k8s
    - Docker
    - Clickhouse
---

> - Create Date: 2019-12-03
> - Update Date: 2021-03-15

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


## prometheus
启动:`./prometheus --config.file=.../prometheus.yml`
开启kafka-exporter
`docker run -ti --rm -p 9308:9308 danielqsj/kafka-exporter --kafka.server=192.168.121.174:9092 --kafka.server=192.168.121.174:9093 --kafka.server=192.168.121.174:9094`

#### 使用Dokcerfile定制镜像
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

- 其他`docker build`的用法
    - 直接使用Git repo进行构建
    `docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14`
    - 用给定的tar包进行构建
    `docker build http://server/context.tar.gz`

- Dockerfile指令详解
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


## docker+clickhouse
### 环境准备
安装docker: `brew install --cask docker`
#### 单机环境准备(macos)
- clickhouse: `docker pull yandex/clickhouse-server`, `docker pull yandex/clickhouse-client`
- 启动容器：`docker run -d -ti --name ck-server  -p 8123:8123 -p 9000:9000 -p 9009:9009 --ulimit nofile=262144:262144  yandex/clickhouse-server`
- 进入容器：`docker exec -ti 845c6ce1ba13 bash`
- 启动ck client: `clickhouse-client`

#### 集群环境准备
一键搭建集群：`git@github.com:yangxkniuniuniu/clickhouse-cluster.git`按照步骤进行搭建
单节点zookeeper，四节点clickhouse集群：包含两个shard，两个replica:
```xml
<remote_servers>
    <company_cluster>
        <shard>
            <replica>
                <host>clickhouse01</host>
                <port>9000</port>
            </replica>
            <replica>
                <host>clickhouse02</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <replica>
                <host>clickhouse03</host>
                <port>9000</port>
            </replica>
            <replica>
                <host>clickhouse04</host>
                <port>9000</port>
            </replica>
        </shard>
    </company_cluster>
  </remote_servers>
```
部署集群后后：
`docker container ls`

![clickhouse集群](https://tva1.sinaimg.cn/large/008eGmZEgy1goeue2r8nqj320c06uaf8.jpg)

`docker exec -ti clickhouse-01 bash`进入

**后续学习均基于clickhouse集群进行**

### clickhouse接口
1 客户端：进入容器后，执行`clickhouse-client`进入（在安装部署后，系统会默认安装）
也可以选择不进入客户端：`cat file.csv | clickhouse-client --database=test --query="INSERT INTO test FORMAT CSV";`

2 http客户端：`echo 'SELECT 1' | curl 'http://localhost:8123/' --data-binary @-`

3 mysql接口：
通过在配置文件中添加`<mysql_port>9004</mysql_port>`，之后执行`mysql --protocol tcp -u default -P 9004`，如果连接成功即可通过mysql进行操作

4 另外clickhouse还提供jdbc/odbc驱动

### clickhouse数据存储与索引机制
#### 数据存储
1. 数据片段datapart
ck表是由**按主键排序的数据片段**组成，数据片段又可以以`Widw`和`Compact`两个格式存储。
`Wide`: 每一列都会存储为单独的文件
`Compact`: 所有列都存储在一个文件中，这种合适可以提高插入量少插入频繁频繁时的性能
存储格式由`min_bytes_for_wide_part`,`min_rows_for_wide_part`控制。如果数据片段中的字节数或行数少于相应的设置值，数据片段会以 `Compact` 格式存储，否则会以 `Wide` 格式存储。

2. 颗粒granules
每个数据片段会被逻辑的分割成颗粒，颗粒是ck进行数据查询时最小不可分割数据集(包含整数个行)

**clickhouse存储接口及索引机制**
> 带着问题往下看：
- MergeTree有哪些文件？
- MergeTree数据如何分布？
- MergeTree索引如何组织？
- MergeTree如何利用索引加速？

准备试验：
```sql
CREATE TABLE company_db.event_test
(
    `time` DateTime,
    `uid` Int64,
    `type` LowCardinality(String),
    INDEX `idx_type` (type) TYPE minmax GRANULARITY 1
)
ENGINE = MergeTree()
PARTITION BY toDate(time)
ORDER BY uid
SETTINGS index_granularity = 3  -- 每3行数据作为一个颗粒
-- 准备数据
INSERT INTO company_db.event_test VALUES
    ('2020-01-01 10:00:00', 100, 'view'),
    ('2020-01-01 10:05:00', 101, 'view'),
    ('2020-01-01 10:06:00', 101, 'contact'),
    ('2020-01-01 10:07:00', 101, 'cancel'),
    ('2020-01-01 10:08:00', 101, 'view'),
    ('2020-01-01 10:09:00', 100, 'cancel'),
    ('2020-01-01 10:10:00', 101, 'view'),
    ('2020-01-01 10:11:00', 103, 'buy'),
    ('2020-01-01 10:12:00', 104, 'buy'),
    ('2020-01-01 10:13:00', 101, 'view'),
    ('2020-01-01 11:14:00', 100, 'contact'),
    ('2020-01-01 12:10:00', 101, 'view'),
    ('2020-01-02 08:10:00', 100, 'view'),
    ('2020-01-03 13:00:00', 103, 'view');
-- 当前目录结构
-- 【20200101_1_1_0  20200102_2_2_0  20200103_3_3_0  detached  format_version.txt】
-- 查看20200101_1_1_0分区数据目录结构
-- ├── checksums.txt
-- ├── columns.txt
-- ├── count.txt
-- ├── minmax_time.idx
-- ├── partition.dat
-- ├── primary.idx
-- ├── skp_idx_idx_type.idx
-- ├── skp_idx_idx_type.mrk2
-- ├── time.bin
-- ├── time.mrk2
-- ├── type.bin
-- ├── type.dict.bin
-- ├── type.dict.mrk2
-- ├── type.mrk2
-- ├── uid.bin
-- └── uid.mrk2
----- 此分区下的数据 -----
-- ┌────────────────time─┬─uid─┬─type────┐
-- │ 2020-01-01 10:00:00 │ 100 │ view    │
-- │ 2020-01-01 10:09:00 │ 100 │ cancel  │
-- │ 2020-01-01 11:14:00 │ 100 │ contact │
-- │ 2020-01-01 10:05:00 │ 101 │ view    │
-- │ 2020-01-01 10:06:00 │ 101 │ contact │
-- │ 2020-01-01 10:07:00 │ 101 │ cancel  │
-- │ 2020-01-01 10:08:00 │ 101 │ view    │
-- │ 2020-01-01 10:10:00 │ 101 │ view    │
-- │ 2020-01-01 10:13:00 │ 101 │ view    │
-- │ 2020-01-01 12:10:00 │ 101 │ view    │
-- │ 2020-01-01 10:11:00 │ 103 │ buy     │
-- │ 2020-01-01 10:12:00 │ 104 │ buy     │
-- └─────────────────────┴─────┴─────────┘
-- 此分区下的数据又会被划分成12/3=4个颗粒
```
解析：
- `*.bin`是列数据文件，按`ORDER BY`排序(uid)
- `*.mrk2`文件用于快速定位bin文件数据位置。会记录每个颗粒的第一行的主键信息：值和offset
- `minmax_time.idx`分区键min-max索引文件，加速分区键查找
- `primary.idx`主键索引文件，加速主键查找
- `skp_idx_idx_type.*`字段type的索引文件，加速type的查找

稀疏索引：
- 主键索引：取每个颗粒的最小值
- 跳数索引：`INDEX `idx_type` (type) TYPE minmax GRANULARITY 1`针对字段type创建了一个minmax模式索引，每1个颗粒选择一个min和一个max
- 分区索引：针对分区键创建minmax索引，加速分区选择

**使用debug模式查看索引机制**
打开debug模式：`clickhouse-client --send_logs_level debug`
索引：
```sql
SELECT *
FROM company_db.event_test
WHERE uid = 101 and view = 'cancel'
<Debug> executeQuery: (from 127.0.0.1:53576) SELECT * FROM company_db.event_test WHERE (type = 'view') AND (uid = 101)
<Debug> InterpreterSelectQuery: MergeTreeWhereOptimizer: condition "type = 'view'" moved to PREWHERE
<Debug> company_db.event_test (SelectExecutor): Key condition: unknown, (column 0 in [101, 101]), and, unknown, and
<Debug> company_db.event_test (SelectExecutor): MinMax index condition: unknown, unknown, and, unknown, and
<Debug> company_db.event_test (SelectExecutor): Index `idx_type` has dropped 1 granules.
<Debug> company_db.event_test (SelectExecutor): Index `idx_type` has dropped 0 granules.
<Debug> company_db.event_test (SelectExecutor): Index `idx_type` has dropped 0 granules.
<Debug> company_db.event_test (SelectExecutor): Selected 3 parts by date, 1 parts by key, 4 marks to read from 1 ranges
┌────────────────time─┬─uid─┬─type─┐
│ 2020-01-01 10:05:00 │ 101 │ view │
│ 2020-01-01 10:08:00 │ 101 │ view │
│ 2020-01-01 10:10:00 │ 101 │ view │
│ 2020-01-01 10:13:00 │ 101 │ view │
│ 2020-01-01 12:10:00 │ 101 │ view │
└─────────────────────┴─────┴──────┘
<Information> executeQuery: Read 12 rows, 221.00 B in 0.007 sec., 1769 rows/sec., 31.82 KiB/sec.
<Debug> MemoryTracker: Peak memory usage (for query): 0.00 B.
-- 结合表中数据看
-- ┌────────────────time─┬─uid─┬─type────┐
-- │ 2020-01-01 10:00:00 │ 100 │ view    │
-- │ 2020-01-01 10:09:00 │ 100 │ cancel  │
-- │ 2020-01-01 11:14:00 │ 100 │ contact │ -- 颗粒1
-- │ 2020-01-01 10:05:00 │ 101 │ view    │
-- │ 2020-01-01 10:06:00 │ 101 │ contact │
-- │ 2020-01-01 10:07:00 │ 101 │ cancel  │ -- 颗粒2
-- │ 2020-01-01 10:08:00 │ 101 │ view    │
-- │ 2020-01-01 10:10:00 │ 101 │ view    │
-- │ 2020-01-01 10:13:00 │ 101 │ view    │ -- 颗粒3
-- │ 2020-01-01 12:10:00 │ 101 │ view    │
-- │ 2020-01-01 10:11:00 │ 103 │ buy     │
-- │ 2020-01-01 10:12:00 │ 104 │ buy     │ -- 颗粒4
-- └─────────────────────┴─────┴─────────┘
-- ┌────────────────time─┬─uid─┬─type─┐
-- │ 2020-01-03 13:00:00 │ 103 │ view │    -- 颗粒5
-- └─────────────────────┴─────┴──────┘
-- ┌────────────────time─┬─uid─┬─type─┐
-- │ 2020-01-02 08:10:00 │ 100 │ view │    -- 颗粒6
-- └─────────────────────┴─────┴──────┘
```
挑重点来看, 数据查询顺序：
1. 分区筛选`Selected 3 parts by date`：检索minmax_time.idx文件，根据分区字段筛选符合要求的数据分片，三个数据分片符合要求
2. 主键筛选`1 parts by key`：`column 0 in [101, 101]`根据primary.idx文件，筛选出一个数据分片
3. `4 marks to read from 1 ranges`：根据uid.mrk2(记录每个颗粒的最小值)，筛选出四个颗粒
4. 在符合条件的四个颗粒中，按照稀疏索引继续查找，仍然有四个颗粒符合要求
`Read 12 rows`：上面四个颗粒有12行数据，按照最后筛选，筛选出5行结果数据
**结论**：所以通过分区+颗粒+索引的方式，对于数据量大的表来说，效率更高

#### 索引
1. 什么时候使用索引：对于`SELECT`查询，下列场景ck会分析是否使用索引，`WHERE/PREWHERE`子句有如下表达式
- 包含一个表示与主键/分区键中的部分字段或全部字段相等/不等的比较表达式
- 基于主键/分区键的字段上的`IN`或固定前缀的`LIKE`表达式
- 基于主键/分区键的字段上的某些函数
- 基于主键/分区键的表达式的逻辑表达式
- 改善索引性能：当前主键是(a,b),在使用c作为查询条件，或者很长数据范围内(a,b)都是相同的值，这两种情况下主键加入c列会提高性能

> 过长的主键会对插入性能和内存消耗有负面影响，但是并不影响select的查询性能

2. 跳数索引
`INDEX index_name expr TYPE type(...) GRANULARITY granularity_value`
可以用来跳过大片不满足条件的数据，减少从磁盘读取的数据量
常用`TYPE`:
- `minmax`: 存储指定表达式的极值，类似于主键
- `set(max_rows)`：存储指定表达式的不重复值(不超过`max_rows`个，`max_rows=0`表示无限制)


### 表引擎
#### 建表语法
```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]
```

- `SAMPLE BY`: 用于抽样的表达式，如果使用的话，主键中必须要包含这个表达式，如：
`SAMPLE BY intHash32(UserId) ORDER BY (CountId, EventDate, intHash32(UserId))`

- `TTL`: 指定行存储的持续时间并定义数据片段在硬盘或卷上的移动逻辑规则列表（可选）
`DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'`

- `SETTINGS`: 控制MergeTree的额外参数
  - `index_granularity` -- 索引粒度。索引中相邻的『标记』间的数据行数。默认值，8192
  - `index_granularity_bytes` -- 索引粒度，以字节为单位，默认值: 10Mb。如果想要仅按数据行数限制索引粒度, 请设置为0(不建议)。
  - `enable_mixed_granularity_parts` -- 与`index_granularity_bytes`搭配使用，如果表里有很大的行，开启这项配置会提升`SELECT`的查询性能
  - `min_merge_bytes_to_use_direct_io` -- 使用直接IO来操作磁盘的合并操作时要求的最小数据量。合并数据片段时，ClickHouse 会计算要被合并的所有数据的总存储空间。如果大小超过设置项则 ClickHouse 将使用直接 I/O 接口（`O_DIRECT` 选项）对磁盘读写。如果设置为0，则直接禁用I/O。默认值：`10 * 1024 * 1024 * 1024` 字节
  - `merge_with_ttl_timeout` --  TTL合并频率的最小间隔时间，单位：秒。默认值: 86400 (1 天)
  - `write_final_mark` -- 是否启用在数据片段尾部写入最终索引标记。默认值: 1（不建议更改）
  - `merge_max_block_size` -- 在块中进行合并操作时的最大行数限制。默认值：8192
  - `storage_policy` -- 存储策略
  - `min_bytes_for_wide_part`,`min_rows_for_wide_part` -- 在数据片段中可以使用Wide格式进行存储的最小字节数/行数。你可以不设置、只设置一个，或全都设置

#### 列TTL和表TTL
- 列TTL：当列中的值过期时，ck会先将他们替换成该列数据类型的默认值，在数据片段中列的所有值均已过期，则会从文件系统中的数据片段删除此列
```sql
-- 创建表时指定 TTL
CREATE TABLE example_table
(
    d DateTime,
    a Int TTL d + INTERVAL 1 MONTH,
    b Int TTL d + INTERVAL 1 MONTH,
    c String
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(d)
ORDER BY d;
-- 为表中已存在的列字段添加 TTL
ALTER TABLE example_table MODIFY COLUMN c String TTL d + INTERVAL 1 DAY;
-- 修改列字段的 TTL
ALTER TABLE example_table MODIFY COLUMN c String TTL d + INTERVAL 1 MONTH;
```

- 表TTL
当表中的行过期时，CK会删除对应的行
TTL 规则的类型紧跟在每个 TTL 表达式后面，它会影响满足表达式时（到达指定时间时）应当执行的操作：
`DELETE` - 删除过期的行（默认操作）;
`TO DISK 'aaa'` - 将数据片段移动到磁盘 aaa;
`TO VOLUME 'bbb'` - 将数据片段移动到卷 bbb.
```sql
-- 创建时指定TTL
CREATE TABLE example_table
(
    d DateTime,
    a Int
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(d)
ORDER BY d
TTL d + INTERVAL 1 MONTH [DELETE],
    d + INTERVAL 1 WEEK TO VOLUME 'aaa',
    d + INTERVAL 2 WEEK TO DISK 'bbb';
-- 修改表的TTL
ALTER TABLE example_table MODIFY TTL d + INTERVAL 1 DAY;
```

- 数据删除：当ck发现数据过期时，将会执行一个计划外的Merge。要控制这类合并的频率, 你可以设置`merge_with_ttl_timeout`。如果该值被设置的太低, 它将引发大量计划外的合并，这可能会消耗大量资源。


#### MergeTree家族
1. MergeTree
`ENGINE = MergeTree()`
同一批次插入的数据会重新按照partition by规则写入新的分区，不会关心之前批次生成的分区。在ck内部会不定时的进行分区merge(大约是插入后15min左右),merge的过程是生成一个新的分区，然后把历史分区标记成(active=0)的状态，过段时间会把active=0的分区删除，再过段时间会删除相应的元数据

2. ReplacingMergeTree
`ENGINE = ReplacingMergeTree([version])` 如果version列未指定，保留最后一条，否则保留version最大的版本
删除排序键值相同的重复项。因为ck的Merge时间是不确定的，所以不能保证表中的数据是完全无重复的

3. CollapsingMergeTree
`ENGINE = CollapsingMergeTree(sign)`
会异步删除Sign有1和-1，且其余字段都一样的成对出现的行

4. VersionedCollapsingMergeTree
`ENGINE = VersionedCollapsingMergeTree(sign, version)`
在CollapsingMergeTree基础上增加Version,符合上面的条件且Version一样的成对出现的行才会被删除（删除动作在Merge后）
适用于：
- 允许快速写入不断变化的对象状态
- 删除后台中的旧对象状态（可降低存储体积）

算法：
- 当ck插入数据时，会按照主键对行进行排序，如果Version不在主键中，ck会隐式的将其添加到主键中并进行排序
- 当ck合并数据部分时，会删除具有相同主键和Version但是Sign不同的行

注意：当在进行查询时，数据合并的时间点无法预知，即数据可能尚未Merge，这时就需要使用聚合，仅对**有效的**数据行进行统计
使用示例：
```sql
-- 创建表
CREATE TABLE UAct
(
  UserID UInt64,
  PageViews UInt8,
  Duration UInt8,
  Sign Int8,
  Version UInt8
)
ENGINE = VersionedCollapsingMergeTree(Sign, Version)
Order by UserID
-- 查询
SELECT
  UserID,
  SUM(PageViews * Sign) AS PageViews,
  SUM(Duration * Sign) AS Duration,
  Version
FROM UAct
GROUP BY UserID, Version
HAVING SUM(Sign) > 0
```

5. SummeringMergeTree
`ENGINE = SummeringMergeTree([columns])`  columns(必须是数值类型)是将要被汇总的列名的元组，未指定columns则默认汇总所有数值类型字段，非数值类型字段随机取一个

6. AggregatingMergeTree
使用此引擎，ck会将一个数据片段内具有相同排序件的行聚合成一行，这一行会存储一系列聚合函数的状态
可用`AggregatingMergeTree`表来做增量数据的聚合统计，包括物化视图的数据聚合
6.1 `ENGINE = AggregatingMergeTree()`
不能够直接用Insert来写入数据，需要结合insert select.
注意：不可以将新增的数据聚合起来
```sql
-- 基础表
create table tb_test_MergeTree_basic
(
  brandId Int32,
  shopId Int32,
  saleDate Date,
  saleMoney Float32,
  saleQty Int32,
  vipId UInt64
)
engine = MergeTree ()
ORDER BY (brandId,shopId)
PARTITION BY (brandId,shopId)
-- 测试数据
insert into tb_test_MergeTree_basic values (429,6001,'2020-10-01 14:15:23',200.50,10,10001),(429,6001,'2020-10-02 14:15:23',200.50,20,10002),(429,6001,'2020-10-03 14:15:23',200.50,30,10003),(429,6001,'2020-10-04 14:15:23',200.50,10,10001),(429,6001,'2020-10-05 14:15:23',200.50,20,10001),(429,6001,'2020-10-06 14:15:23',200.50,30,10003),(429,6002,'2020-10-04 14:15:23',200.50,40,10001),(429,6002,'2020-10-05 14:15:23',200.50,10,10001)
-- 使用聚合树引擎表
CREATE TABLE tb_test_AggregatingMergeTree_table
(
    `brandId` Int32,
    `shopId` Int32,
    `saleMoney` AggregateFunction(sum, Float32),
    `saleQty` AggregateFunction(sum, Int32),
    `saleNum` AggregateFunction(count, UInt8),
    `vipNum` AggregateFunction(uniq, UInt64)
)
ENGINE = AggregatingMergeTree()
PARTITION BY (brandId, shopId)
ORDER BY (brandId, shopId)
-- 插入数据时需要结合聚合函数
INSERT INTO tb_test_AggregatingMergeTree_table
SELECT
    brandId,
    shopId,
    sumState(saleMoney) AS saleMoney,
    sumState(saleQty) AS saleQty,
    countState(1) AS saleNum,
    uniqState(vipId) AS vipNum
FROM tb_test_MergeTree_basic
GROUP BY
    brandId,
    shopId
-- 查询的时候也要加聚合函数
SELECT
    brandId,
    shopId,
    sumMerge(saleMoney) AS saleMoney,
    sumMerge(saleQty) AS saleQty,
    countMerge(saleNum) AS saleNum,
    uniqMerge(vipNum) AS vipNum
FROM tb_test_AggregatingMergeTree_table
GROUP BY
    brandId,
    shopId
```

6.2 AggregatingMergeTree物化视图
```sql
CREATE MATERIALIZED VIEW tb_test_AggregatingMergeTree_view
ENGINE = AggregatingMergeTree()
PARTITION BY (brandId, shopId)
ORDER BY (brandId, shopId) AS
SELECT
    brandId,
    shopId,
    sumState(saleMoney) AS saleMoney,
    sumState(saleQty) AS saleQty,
    countState(1) AS saleNum,
    uniqState(vipId) AS vipNum
FROM tb_test_MergeTree_basic
GROUP BY
    brandId,
    shopId
-- 创建物化视图前的数据已经不能再被跟踪
-- 继续往基础表插入数据
insert into tb_test_MergeTree_basic values (429,6001,'2020-10-08 14:15:23',200.50,30,10003),(429,6002,'2020-10-08 14:15:23',200.50,40,10002)
-- 数据已经自动跟踪了，如下
-- ┌─brandId─┬─shopId─┬─saleMoney─┬─saleQty─┬─saleNum─┬─vipNum─┐
-- │     429 │   6001 │ i@        │         │         │ Gw     │
-- └─────────┴────────┴───────────┴─────────┴─────────┴────────┘
-- ┌─brandId─┬─shopId─┬─saleMoney─┬─saleQty─┬─saleNum─┬─vipNum─┐
-- │     429 │   6002 │ i@        │ (       │         │ $a6    │
-- └─────────┴────────┴───────────┴─────────┴─────────┴────────┘
select
  brandId,
  shopId,
  sumMerge(saleMoney) as saleMoney,
  sumMerge(saleQty) as saleQty,
  countMerge(saleNum) as saleNum,
  uniqMerge(vipNum) as vipNum
from tb_test_AggregatingMergeTree_view
group by brandId,shopId
```
