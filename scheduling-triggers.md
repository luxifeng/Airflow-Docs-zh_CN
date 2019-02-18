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



