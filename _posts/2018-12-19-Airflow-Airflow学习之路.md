---
layout:     post
title:      Git 从踩坑到不停踩坑
subtitle:   git
date:       2018-12-19
author:     owl city / covered from BY
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - Mac
    - 终端
    - Git
---


# airflow
安装：`sudo pip install apache-airflow==1.9.0 --ignore-installed python-dateutil`
### quickstart
`airflow initdb`  初始化数据库
`airflow webserver -p 8080` 开启网页端服务
`airflow scheduler`  开启调度器

###编写调度程序
- 导入依赖
```
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
```
- 设置参数
```
from datetime import datetime, timedelta

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2015, 6, 1),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
}
```
- 实例化一个DAG
```
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag
)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag
)
```
- 建立依赖关系
`t2.set_upstream(t1)`等价于`t1.set_downstream(t2)`

### Testing
模拟在某个时间段执行这个task
`airflow test airtest start 2018-10-25`
### backfill
回填,重新执行start->end这段时间的dag
`airflow backfill airtest -s 2018-10-22 -e 2018-10-25`

### Operator
#### BashOperator
```
run_this = BashOperator(
    task_id='run',
    bash_command='echo 0',
    dag=dag
)
```
#### PythonOperator
```
def my_python(**kwargs):
    logging.info('I am logging...')
    print('I am printing...')
    print(kwargs['params']['aa'])

t2 = PythonOperator(
    task_id='python_test',
    provide_context=True,
    python_callable=my_python,
    params={
        'aa': '1',
        'bb': '2'
    },
    dag=dag
)
```

### Variable & Connection
```
from airflow.models import Variable, Connection

# get Variable
env = Variable.get('airflow_env')

# get Connection
session = settings.Session()
connect = session.query(Connection).filter(Connection.conn_id == 'local_mysql').first()
```



### Scheduling & Triggers
`airflow trigger_dag dag_id`

## java解析json文件（org.josn和gson）
### org.json
- 读取json
```
String json = "{\"name\":\"Tom\",\"age\":\"12\",\"info\":{\"id\":\"144\",\"uuid\":112}}";
JSONObject jsonObject = new JSONObject(json);
String name = jsonObject.getString("name");
String age = jsonObject.getString("age");
JSONObject object1 = jsonObject.getJSONObject("info");
String id = object1.getString("id");
Integer uuid = object1.getInt("uuid");
System.out.println("name:"+name+"...age:"+age+"...info:>id:"+id+"..>uuid:"+uuid);
```
- 生成json
```
JSONObject jsonObject1 = new JSONObject();
jsonObject1.put("title","数学");
jsonObject1.put("score",90);
System.out.println(jsonObject1.toString());
```
> 也可以使用JSONArray的方式读取和生成json
> [详细使用方法](https://blog.csdn.net/Zen99T/article/details/50351637)
