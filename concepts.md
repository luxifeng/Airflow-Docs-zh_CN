# 概念

Airflow平台是一个用于描述、执行和监控工作流的工具。

### 核心概念

#### 有向无环图（DAGs）

在Airflow中，一个`DAG`——或者说有向无环图（Directed Acyclic Graph）——是你想要运行的所有任务集合，以一种能反映它们的关系和依赖的方式组织在一起。

例如，一个简单的DAG可以由三个任务组成：A，B和C。可以让任务A运行成功后任务B才能运行，而任务C可以在任何时候运行。可以让任务A运行5分钟后超时，以及任务B若失败了能最多重启5次。也可以让工作流在每晚10点运行，但直至某个日期才启动。

这样，一个DAG描述了你希望_如何_执行你的工作流；但是注意一下，我们从未说过我们真正想做_什么_！A，B和C可以是任何事。可能A是准备数据给B分析，而C是发送邮件。也可能A是监控你的位置，这样B可以打开你的车库门，而C是打开你家的灯。重要的事情是，DAG不关心它的组成任务是做什么的；它的工作是确保任务要做的事情发生在正确的时间，或正确的顺序，或能够正确地处理异常问题。

DAG由标准的Python文件所定义，被放置在Airflow的`DAG_FOLDER`文件夹。Airflow会执行每个文件的代码，动态地构建`DAG`对象。你可以拥有任意数量的DAG，每个DAG可以描述任意数量的任务。一般来说，每个DAG应当对应单个逻辑工作流。

{% hint style="info" %}
搜寻DAG时，Airflow只会考虑那些内容中同时出现了字符串“airflow”和“DAG”的`.py`文件。
{% endhint %}

### 作用域（Scope）

Airflow会加载任何能从DAG文件导入的`DAG`对象。这就要求DAG必须存在于`globals()`中。考虑下列两个DAG。只有dag\_1会被加载；另一个只会出现在局部作用域（local scope）。

```python
dag_1 = DAG('this_dag_will_be_discovered')

def my_function():
    dag_2 = DAG('but_this_dag_will_not')

my_function()
```

有时候这可以派上用场。例如，SubDagOperator的一个通用模式是定义方法中的子dag，使得Airflow不会试图将它当成独立的DAG进行加载。

### 默认参数（Default Arguments）

如果一个`default_args`字典传递给了DAG，就会被应用到该DAG的所有operator中。这样一个共同的参数可以应用到多个operator，而不必多次输入。

```python
default_args = {
    'start_date': datetime(2016, 1, 1),
    'owner': 'Airflow'
}

dag = DAG('my_dag', default_args=default_args)
op = DummyOperator(task_id='dummy', dag=dag)
print(op.owner) # Airflow
```

### 上下文管理器（Context Managers）

_Airflow 1.8中增加_

DAG可以用作上下文管理器，自动分配新的operator给那个DAG。

```python
with DAG('my_dag', start_date=datetime(2016, 1, 1)) as dag:
    op = DummyOperator('op')

op.dag is dag # True
```

### 操作（Operators）

DAG描述如何运行一个工作流，`Operators`则决定实际做什么。

一个operator描述了一个工作流中的一个单独的任务。Operator们通常（并非总是）是原子的，即它们是独立存在的，不需要与其他operator分享资源。DAG会确保operator们以正确的顺序运行；与那些依赖项不同，operator们通常独立运行。事实上，它们可能运行在两个完全不同的机器上。

这是一个微妙但非常重要的点：一般而言，如果两个operator需要共享信息，如一个文件名或少量数据，你应考虑将它们结合成一个单独的operator。如果分开绝不能避免，Airflow也有一个特性名为XCom，可供operator们交叉通信，本文档的其他部分对其进行了描述。

Airflow提供了很多operator，应付许多常用的任务，包括：

* `BashOperator`- 执行一个bash命令
* `PythonOperator`- 调用任意Python方法
* `EmailOperator`- 发送一封邮件
* `SimpleHttpOperator`- 发送一个HTTP请求
* `MySqlOperator`, `SqliteOperator`, `PostgresOperator`, `MsSqlOperator`, `OracleOperator`, `JdbcOperator`等 - 执行一个SQL命令
* `Sensor`- 等待某个时间、文件、数据库行记录、S3 key等

除了这些基本构件之外，还有超多特定的operator：`DockerOperator`, `HiveOperator`, `S3FileTransformOperator`, `PrestoToMysqlOperator`, `SlackOperator`……你懂的！

`airflow/contrib/`目录包含了更多由社区产生的operator。这些operator并不总是像主要发行版本里的operator那样完整和充分测试，但是方便了用户给平台添加新的功能。

只有当operator分配给了DAG，Airflow才会加载它们。

访问[使用Operator](https://airflow.apache.org/howto/operator.html)查看如何使用Airflow operators。

