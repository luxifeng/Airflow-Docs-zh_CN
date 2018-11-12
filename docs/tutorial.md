# 教程

这篇教程会指导你编写第一条管道，了解基本的Airflow概念、对象和它们的用法。

## 管道定义示例

这里有一个基本的管道定义示例。不要担心它看上去挺复杂，逐行解释如下。

```python
"""
Code that goes along with the Airflow tutorial located at:
https://github.com/apache/incubator-airflow/blob/master/airflow/example_dags/tutorial.py
"""
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
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

dag = DAG('tutorial', default_args=default_args)

# t1, t2 and t3 are examples of tasks created by instantiating operators
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag)

templated_command = """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
        echo "{{ params.my_param }}"
    {% endfor %}
"""

t3 = BashOperator(
    task_id='templated',
    bash_command=templated_command,
    params={'my_param': 'Parameter I passed in'},
    dag=dag)

t2.set_upstream(t1)
t3.set_upstream(t1)
```

## 它是DAG定义文件

上面的Airflow Python脚本实际上只是一个配置文件，用代码来指定DAG结构，这一点可以让你的思路清晰一些（可能不是每个人都觉得很直观）。定义的任务会在不同的上下文中运行，并且上下文与脚本有关。不同的任务在不同的时间点运行在不同的工作节点上，这意味着脚本不适用于任务间的交叉通信。请注意，为了实现交叉通信，我们有一个更高级的特性，叫做`XCom`。

有时候人们认为这个DAG定义文件是用来实现一些实际的数据处理，但事实并非如此！此脚本的目的是定义DAG对象。它需要快速求值（秒级，非分钟级），因为调度程序会周期性执行它，来反映任何的变化。

## 导入模块

一条Airflow管道只是一个用来定义Airflow DAG对象的Python脚本。让我们从导入需要的包开始。

```python
# The DAG object; we'll need this to instantiate a DAG
from airflow import DAG

# Operators; we need this to operate!
from airflow.operators.bash_operator import BashOperator
```

## 默认参数

我们即将创建一个DAG和一些任务，并且我们可以选择显式地将一组参数传递给每个任务的构造函数（冗余）或者（更佳的选择）我们可以定义用于创建任务的默认参数字典。

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

想了解更多关于BaseOperator的参数和它们的作用，请参考`airflow.models.BaseOperator`文档。

同样，请注意你可以方便地定义用于不同目的的不同参数集。一个例子是，在生产和开发环境中使用不同的设置。

## 初始化DAG

我们需要一个DAG对象来潜入我们的任务。在此我们传递了一个定义`dag_id`的字符串，用于唯一识别DAG对象。同时我们传递了刚定义的默认参数字典和值为1天的`schedule_interval`给DAG对象。

```python
dag = DAG(
    'tutorial', default_args=default_args, schedule_interval=timedelta(1))
```

## 任务

任务是在实例化操作符对象时生成的。从运算符实例化的对象称为构造函数。第一个参数`task_id`是任务的唯一标识。

```python
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag)
```

注意我们是如何混合传递特定参数（`bash_command`）和继承自BaseOperator的共同参数（`retries`）给操作符的构造函数的。这比传递每一个参数给每一个构造函数简单多了。同时，注意在第二个任务中，我们用值`3`重写了`retries`。

任务的优先规则如下：

1. 明确传递的参数
2. 存在在`default_args`字典中的值
3. 若存在，操作符的默认值

任务必须包含或继承参数`task_id`和`owner`，否则Airflow会抛出异常。

## 使用Jinja制作模板

Airflow借助[Jinja模板](http://jinja.pocoo.org/docs/dev/)的力量，提供了一系列内置参数和宏给管道编辑者。Airflow也提供了钩子（hook）给管道编辑者，来定义他们自己的参数、宏和模板。

这篇教程很少涉及到关于制作模板的内容，但是这一小节的出现是为了让你知道这个特性是存在的，帮助你熟悉双花括号的用法，并且指明最常用的模板变量：`{{ ds }}`（当天日期戳）。

```python
templated_command = """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7) }}"
        echo "{{ params.my_param }}"
    {% endfor %}
"""

t3 = BashOperator(
    task_id='templated',
    bash_command=templated_command,
    params={'my_param': 'Parameter I passed in'},
    dag=dag)
```

请注意 `templated_command`把代码逻辑包含在`{% %}`块中，引用参数如`{{ ds }}`，调用函数如`{{ macros.ds_add(ds, 7)}}`，引用用户定义参数如`{{ params.my_param }}`。

`BaseOperator`中的`params`钩子允许你将参数和/或对象的字典传递给模板。请花些时间理解参数`my_param`是如何传递给模板的。

也可以将文件传递给`bash_command`参数，像`bash_command='templated_command.sh'`，文件位置是包含管道文件的目录的相对路径（本例是`tutorial.py`）。这样做有很多理由，比如分离脚本中的逻辑和管道代码、允许高亮不同语言组成的文件中的正确代码，以及保持结构化管道的整体灵活度。还可以在DAG构造函数中定义你的`template_searchpath`指向任何文件夹位置。

