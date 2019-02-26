# 调度与触发

Airflow调度器监控所有的任务和所有的DAG，并且触发依赖条件已满足的任务实例。在背后，它启动子进程——子进程监控一个文件夹下可能包含的所有DAG对象，且与该文件夹保持同步——并周期性地（分钟左右）收集DAG解析结果和检查活跃任务以了解它们能否被触发。

Airflow调度器被设计为持久服务运行于Airflow生产环境中。为启动调度器，你仅需执行`airflow scheduler`。它会使用`airflow.cfg`中指定的配置。

请注意，如果你以一天的`schedule_interval`运行DAG，那么`2016-01-01T23:59`后很快就会跑标记为`2016-01-01`的运行。换言之，一旦配置覆盖的时间过了，作业实例就会启动。

再说一遍，调度器以启动时间后的一个`schedule_interval`运行你的作业，在周期的最后。

调度器会启动你的`airflow.cfg`中指定的执行器实例。如果执行器是`LocalExecutor`，任务会以子进程执行；如果执行器是`CeleryExecutor`和`MesosExecutor`，任务会远程执行。

要启动一下调度器，只需执行命令：

```text
airflow scheduler
```

### DAG Runs

一次DAG运行是一个对象，表示一个时间维度的DAG实例。

每个DAG可能有也可能没有调度计划，可以告知`DAG Runs`如何被创建。`schedule_interval`被定义为DAG参数，可以优先以`str`类型接收[cron表达式](https://en.wikipedia.org/wiki/Cron#CRON_expression)，或者一个`datetime.timedelta`对象。或者，你也可以使用其中一个cron “预设”：

| preset | meaning | cron |
| :--- | :--- | :--- |
| `None` | 勿调度，专为“外部触发”的DAG所用 |  |
| `@once` | 只调度一次 |  |
| `@hourly` | 每小时开始时运行一次 | `0 * * * *` |
| `@daily` | 每天午夜运行一次 | `0 0 * * *` |
| `@weekly` | 每周周日午夜运行一次 | `0 0 * * 0` |
| `@monthly` | 每月第一天午夜运行一次 | `0 0 1 * *` |
| `@yearly` | 每年1月1日午夜运行一次 | `0 0 1 1 *` |

**请注意：**若不希望调度你的DAG，使用`schedule_interval=None`而非`schedule_interval='None'`。

每一次调度，你的DAG都会被实例化，并且会创建一个`DAG Run`实体。

DAG运行拥有一个与之关联的状态（运行中、失败、成功），并通知调度进程哪一个调度计划集可做任务提交评估。没有DAG运行层面的元数据的话，Airflow调度进程得做更多的事情，以找出哪些任务应被触发并有所进展。它也可能会在DAG变形过程中创造不需要的处理，比如说添加新任务。



