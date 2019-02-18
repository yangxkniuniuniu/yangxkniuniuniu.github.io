---
layout:     post
title:      kafka汇报
subtitle:   kafka
date:       2019-02-18
author:     owl city
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - kafka
---

### Kafka     v1.1.1

kafka服务接口地址: beta-tcp-endpoint.dev.saybot.net:31090, 31091, 31092

[项目地址](https://git.saybot.net/cody.yang/k8s-kafka)

![pods](https://ws4.sinaimg.cn/large/006tKfTcgy1g0adlmz8l1j30d903vaa9.jpg)

[Grafana地址](https://grafana-k8s.dev.saybot.net/d/q3Mfvnhmk/kafka-metrics-monitor?orgId=1&from=now-2d&to=now)

[prometheus地址](http://prometheus-k8s.dev.saybot.net/graph)

#### 待解决问题或优化方案:
- 待解决问题: kafka-manager连接问题

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0adoxr4x9j30j70bqab7.jpg)


- Grafana模板显示优化.根据实际情况完成布局显示.

- 变量优化: 资源大小以及副本数量等写入到CI environment variables中去.

- kafka日志查看,接入kibana?




### 数据归档
对Mysql表数据进行定期备份:表中只存放近7天的记录,7天前的记录先插入备份表中,再从表中删除.

流程:

1. 先判断是否创建备份表:  `CREATE TABLE IF NOT EXISTS log.sessions_201901 like log.sessions;`

2. 数据插入备份表: `INSERT INTO log.sessions_201901 SELECT * FROM log.sessions WHERE created_at < '2019-01-21';`

3. 删除原数据: `DELETE FROM log.sessions WHERE id <= (SELECT MAX(id) FROM log.sessions_201901);`



**目标 :  在删除数据的同时不影响数据的查询**

**问题:**
测试用例: 往sessions表中插入日期为2019-01-20的100万条记录,日期为2019-01-21的100条记录

- 测试数据插入

```java
while (i <= 1000000) {
    String sql = "insert into sessions(created_at) values (\"2019-01-20 09:33:32\")";
    st.executeUpdate(sql);
    i++;
    System.out.println("第" + i + "条");
}
```

- 测试数据查询

```java
while (true) {
    Thread.sleep(400);
    ResultSet rs = st.executeQuery("select id from sessions where created_at > \'2019-01-21\';");
    while (rs.next()) {
        int id = rs.getInt(1);
        System.out.println("id: " + id);
    }
}
```



- 在repeatable read 隔离级别下, jdbc先循环查询日期为2019-01-21数据, 再删除日期为2019-01-20的记录.

    - 删除操作等待,停止查询,删除操作能够进行


- 切换为read committed隔离级别,再进行相同操作,删除操作未等待
