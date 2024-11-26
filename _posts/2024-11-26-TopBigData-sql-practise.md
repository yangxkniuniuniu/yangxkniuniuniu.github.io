---
layout:     post
title:      SQL
subtitle:   sql练习
date:       2024-11-26
author:     owl city
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - HIVE
    - SparkSQL
    - 面试
---

> - Create Date: 2024-11-26
> - Update Date: 2024-11-26

# SQL
## 题目1 日期交叉问题

> 如下为某平台的商品促销数据，字段含义分别为品牌名称、打折开始日期、打折结束日期，现在要计算每个品牌的打折销售天数（注意其中的交叉日期）。比如vivo的打折销售天数就为17天。
```sql
with tmp as (
    select 'xiaomi' as brand   ,'2021-06-05' as start_date,'2021-06-09' as end_date
    union all
    select 'xiaomi' as brand   ,'2021-06-11' as start_date,'2021-06-21' as end_date
    union all
    select 'vivo' as brand   ,'2021-06-05' as start_date,'2021-06-15' as end_date
    union all
    select 'vivo' as brand   ,'2021-06-09' as start_date,'2021-06-21' as end_date
    union all
    select 'honor' as brand  ,'2021-06-05' as start_date,'2021-06-21' as end_date
    union all
    select 'honor' as brand  ,'2021-06-09' as start_date,'2021-06-15' as end_date
    union all
    select 'honor' as brand  ,'2021-06-17' as start_date,'2021-06-26' as end_date
    union all
    select 'huawei' as brand ,'2021-06-05' as start_date,'2021-06-26' as end_date
    union all
    select 'huawei' as brand ,'2021-06-09' as start_date,'2021-06-15' as end_date
    union all
    select 'huawei' as brand ,'2021-06-17' as start_date,'2021-06-21' as end_date
)
select
    brand
    ,sum(datediff(end_date, start_date) + 1) as diff_days
from (
    select
        brand
        ,if(start_date <= max_date, date_add(max_date, 1), start_date) as start_date
        ,end_date
    from (
        select
            brand
            ,start_date
            ,end_date
            ,max(end_date) over(partition by brand order by start_date rows between unbounded preceding and 1 preceding) as max_date
        from tmp
    )
)
where end_date > start_date
group by
    brand
```

## 题目2 同时最大在线人数问题

> 如下为某直播平台各主播的开播及关播时间明细数据，现在需要计算出该平台最高峰期同时在线的主播人数

```sql
with tmp as(
    select 1001 as user_id, '2021-06-14 12:12:12' as start_date , '2021-06-14 18:12:12' as end_date
    union all
    select 1003 as user_id, '2021-06-14 13:12:12' as start_date , '2021-06-14 16:12:12' as end_date
    union all
    select 1004 as user_id, '2021-06-14 13:15:12' as start_date , '2021-06-14 20:12:12' as end_date
    union all
    select 1002 as user_id, '2021-06-14 15:12:12' as start_date , '2021-06-14 16:12:12' as end_date
    union all
    select 1005 as user_id, '2021-06-14 15:18:12' as start_date , '2021-06-14 20:12:12' as end_date
    union all
    select 1001 as user_id, '2021-06-14 20:12:12' as start_date , '2021-06-14 23:12:12' as end_date
    union all
    select 1006 as user_id, '2021-06-14 21:12:12' as start_date , '2021-06-14 23:15:12' as end_date
    union all
    select 1007 as user_id, '2021-06-14 22:12:12' as start_date , '2021-06-14 23:10:12' as end_date
)
select
    user_id
    ,dt
    ,sum(tp) over(order by dt) as num
from (
    select
        user_id
        ,start_date as dt
        ,1 as tp
    from tmp
    union all
    select
        user_id
        ,end_date as dt
        ,-1 as tp
    from tmp
)
```

## 题目3 滑动窗口问题

> 下面是某电商网站的订单数据，包括order_id,user_id,order_status和operate_time四个字段，我们需要找出所有恶意购买的用户。恶意购买的用户定义是：同一个用户，在任意半小时内（含），取消订单次数>=3次的就被视为恶意买家。比如该样例数据中c用户就是恶意买家

```sql
with tmp as(
    select 1101 as order_id, 'a' as user_id, '已支付' as order_status, '2023-01-01 10:00:00' as operate_time
    union all
    select 1102 as order_id, 'a' as user_id, '已取消' as order_status, '2023-01-01 10:10:00' as operate_time
    union all
    select 1103 as order_id, 'a' as user_id, '待支付' as order_status, '2023-01-01 10:20:00' as operate_time
    union all
    select 1104 as order_id, 'b' as user_id, '已取消' as order_status, '2023-01-01 10:30:00' as operate_time
    union all
    select 1105 as order_id, 'a' as user_id, '待确认' as order_status, '2023-01-01 10:50:00' as operate_time
    union all
    select 1106 as order_id, 'a' as user_id, '已取消' as order_status, '2023-01-01 11:00:00' as operate_time
    union all
    select 1107 as order_id, 'b' as user_id, '已取消' as order_status, '2023-01-01 11:40:00' as operate_time
    union all
    select 1108 as order_id, 'b' as user_id, '已取消' as order_status, '2023-01-01 11:50:00' as operate_time
    union all
    select 1109 as order_id, 'b' as user_id, '已支付' as order_status, '2023-01-01 12:00:00' as operate_time
    union all
    select 1110 as order_id, 'b' as user_id, '已取消' as order_status, '2023-01-01 12:11:00' as operate_time
    union all
    select 1111 as order_id, 'c' as user_id, '已取消' as order_status, '2023-01-01 12:20:00' as operate_time
    union all
    select 1112 as order_id, 'c' as user_id, '已取消' as order_status, '2023-01-01 12:30:00' as operate_time
    union all
    select 1113 as order_id, 'c' as user_id, '已取消' as order_status, '2023-01-01 12:55:00' as operate_time
    union all
    select 1114 as order_id, 'c' as user_id, '已取消' as order_status, '2023-01-01 13:00:00' as operate_time
)
select
    user_id
from (
    select
        order_id
        ,user_id
        ,count(order_id) over(partition by user_id order by ts range between 30 * 60 preceding and current row) as cancel_cnt
    from (
        select
            order_id
            ,user_id
            ,unix_timestamp(operate_time) as ts
        from tmp
        where order_status = '已取消'
    )
)
where cancel_cnt >= 3
group by
    user_id
```

## 题目4 间隔连续问题

> 下面是某游戏公司记录的用户每日登录数据, 计算每个用户最大的连续登录天数，定义连续登录时可以间隔一天。举例：如果一个用户在 1,3,5,6,9 登录了游戏，则视为连续 6 天登录。

```sql
with tmp as(
    select 1001 as id,'2021-12-12' as dt
    union all
    select 1002 as id,'2021-12-12' as dt
    union all
    select 1001 as id,'2021-12-13' as dt
    union all
    select 1001 as id,'2021-12-14' as dt
    union all
    select 1001 as id,'2021-12-16' as dt
    union all
    select 1002 as id,'2021-12-16' as dt
    union all
    select 1001 as id,'2021-12-19' as dt
    union all
    select 1002 as id,'2021-12-17' as dt
    union all
    select 1001 as id,'2021-12-20' as dt
)
select
    id
    ,max(days) as max_days
from (
    select
        id
        ,tp
        ,datediff(max(dt), min(dt)) + 1 as days
    from (
        select
            id
            ,dt
            ,sum(if(datediff(dt, last_dt) <= 2, 0, 1)) over(partition by id order by dt) as tp
        from (
            select
                id
                ,dt
                ,lag(dt, 1, dt) over(partition by id order by dt) as last_dt
            from tmp
        )
    )
    group by
        id
        ,tp
)
group by
    id
```

## 题目5 有效值追溯
> 现在有一张商品入库表，包括商品id、商品成本和入库日期3个字段，由于某些原因，导致部分商品的成本缺失（为0或者没有值都是缺失），这样不利于我们计算成本。所以现在要把缺失的商品进价补充完整，补充的依据是使用相同商品的最近一次有效成本作为当前商品的成本。比如2023-11-04号101商品的cost就需要用300.39填充

```sql
with tmp as (
    select 101 as product_id,300.39 as cost,'2023-11-01' as date
    union all
    select 102 as product_id,500 as cost,'2023-11-02' as date
    union all
    select 101 as product_id,0 as cost,'2023-11-03' as date
    union all
    select 101 as product_id,null as cost,'2023-11-04' as date
    union all
    select 102 as product_id,600 as cost,'2023-11-04' as date
    union all
    select 102 as product_id,null as cost,'2023-11-05' as date
    union all
    select 103 as product_id,983 as cost,'2023-11-06' as date
)
select
    product_id
    ,last_value(cost, true) over(partition by product_id order by date) as last_cost
    ,date
from (
    select
        product_id
        ,if(cost = 0.0, null, cost) as cost
        ,date
    from tmp
)
```

## 题目6 分钟级的趋势图

> 在Hive或者ODPS中，怎么用sql实现分钟级的趋势图？比如从交易表中，如何统计0点到每分钟的交易趋势图？原表：trade_A(trade_id，pay_time(格式是2020-08-05 10:30:28)，pay_gmv)。希望用sql实现分钟级的0点到当前分钟的GMV。结果表：result_A(minute_rn(分钟顺序),pay_gmv_td(每分钟的交易额，都是0点到当前分钟的累加值))。

```sql
with tmp as (
    select 101 as trade_id,'2020-08-05 00:30:28' as pay_time,100 as pay_gmv
    union all
    select 102 as trade_id,'2020-08-05 00:30:58' as pay_time,200 as pay_gmv
    union all
    select 103 as trade_id,'2020-08-05 00:35:28' as pay_time,300 as pay_gmv
    union all
    select 104 as trade_id,'2020-08-05 01:36:28' as pay_time,400 as pay_gmv
    union all
    select 105 as trade_id,'2020-08-06 00:20:28' as pay_time,500 as pay_gmv
    union all
    select 106 as trade_id,'2020-08-06 00:21:28' as pay_time,600 as pay_gmv
)
select
    t1.pay_time
    ,sum(nvl(t2.pay_gmv, 0)) over(partition by from_unixtime(unix_timestamp(t1.pay_time), 'yyyy-MM-dd') order by t1.pay_time) as pay_gmv
from (
    select
        from_unixtime(unix_timestamp(min_pay_time) + tab.pos * 60) as pay_time
    from (
        select
            min(from_unixtime(unix_timestamp(pay_time), 'yyyy-MM-dd 00:00:00')) as min_pay_time
            ,max(from_unixtime(unix_timestamp(pay_time), 'yyyy-MM-dd HH:mm:00')) as max_pay_time
        from tmp
    )
    lateral view posexplode(split(repeat(',', (unix_timestamp(max_pay_time) - unix_timestamp(min_pay_time)) / 60), '')) tab as pos, val
) t1
left join (
    select
        trade_id
        ,from_unixtime(unix_timestamp(pay_time), 'yyyy-MM-dd HH:mm:00') as pay_time
        ,pay_gmv
    from tmp
) t2
on t1.pay_time  = t2.pay_time
```
