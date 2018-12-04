# 使用Dask横向扩展

`DaskExecutor`允许你在Dask分布式集群上运行Airflow任务。

Dask集群可以运行在单台机器上或在远程网络上。请翻阅[Distributed文档](https://distributed.readthedocs.io/)获取更完整的细节。

要创建集群，首先须启动一个调度程序：

```bash
# default settings for a local cluster
DASK_HOST=127.0.0.1
DASK_PORT=8786

dask-scheduler --host $DASK_HOST --port $DASK_PORT
```

其次启动至少一个worker，可以是任何能够连接Dask主机的机器：

```bash
dask-worker $DASK_HOST:$DASK_PORT
```

编辑你的`airflow.cfg`，设置执行器为`DaskExecutor`，在`[dask]`小节提供Dask调度程序地址。

请注意：

* 每一个Dask worker必须能够导入airflow及所需的依赖；
* Dask不支持队列。如果一个Airflow任务带着队列创建，那么会出现警告，但任务仍会被提交到集群。

