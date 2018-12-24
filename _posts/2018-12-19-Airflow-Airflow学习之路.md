---
layout:     post
title:      Airflow 学习之路
subtitle:   Airflow
date:       2018-12-19
author:     owl city
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - Airflow
    - Python
    - 终端
---


# airflow
安装：`sudo pip install apache-airflow==1.9.0 --ignore-installed python-dateutil`

## quickstart
### 开启airflow服务
`airflow initdb`  初始化数据库
`airflow webserver -p 8080` 开启网页端服务
`airflow scheduler`  开启调度器
`ps aux | grep airflow` 查看airflow webserver和schedual是否正常启动

###编写调度程序
- 导入依赖

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
```

- 设置参数

```python
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

```python
dag = DAG(
    'user_active_log_etl',
    schedule_interval = '0 2 * * *',
    start_date = datetime(2018, 12, 11, 2),
    default_args = default_args,
    max_active_runs = 1
  )
```

- 定义task

```python
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
> 在新版的airflow中可以使用 `t1 > t2`来控制工作流的顺序
### 名词解释
`depends_on_past`:本次调度任务是否会依赖上一次任务的完成
`start_date`: airflow调度的开始时间
`schedual_interval`: 调度任务的时间
`max_active_runs`: 设置任务跑失败之后的重试次数

### Testing
模拟在某个时间段执行这个task
`airflow test airtest start '2018-10-25 02:00:00'`
### backfill
- 回填,重新执行start->end这段时间的dag
`airflow backfill -s '2018-10-22' -e '2018-10-25' airflowtest`
- 将之前的任务backfill为success
`airflow backfill -s '2018-10-22' -e '2018-10-25' -m -I airflowtest`
- 也可以指定task
`airflow backfill -s '2018-10-22' -e '2018-10-25' -m -I -t start airflowtest`
> - -s为开始时间
> - -e 为结束时间
> - -m make success
> - -I 忽略依赖
> - -t task_regex 指定需要backfill的task

### Operator
#### BashOperator
```python
run_this = BashOperator(
    task_id='run',
    bash_command='echo 0',
    dag=dag
)
```
#### PythonOperator
```python
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
```python
from airflow.models import Variable, Connection

# get Variable
env = Variable.get('airflow_env')

# get Connection
session = settings.Session()
connect = session.query(Connection).filter(Connection.conn_id == 'local_mysql').first()
```



### Scheduling & Triggers
`airflow trigger_dag dag_id` 将会去启动一个dag
- 可以使用-e指定执行时间
- 使用-c传入参数
