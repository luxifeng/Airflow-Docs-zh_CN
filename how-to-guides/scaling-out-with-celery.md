# 使用Celery横向扩展

`CeleryExecutor`是横向扩展worker数量的方式之一。为使其可行，你需要设置Celery后端（RabbitMQ、Redis等），并且修改`airflow.cfg`，将executor参数为`CeleryExecutor`，同时提供相关的Celery设置。

关于Celery代理（Broker）的设置，更多信息请访问详细的[Celery文档](http://docs.celeryproject.org/en/latest/getting-started/brokers/index.html)。

以下是对于worker的必要要求：

* 已安装`airflow`，命令行已在环境变量中配置
* 集群中Airflow的配置应保持一致
* worker的环境有operator的依赖包/组件。比如使用`HiveOperator`，那么环境中应安装hive命令行接口；使用`MySqlOperator`，那么相关的python包应存在在`PYTHONPATH`路径下。
* worker应该能触达`DAGS_FOLDER`，并且你需要通过自己的方式在worker间同步文件系统。常见的设置是将 DAGS\_FOLDER存储在Git库中，并使用Chef、Puppet、Ansible或其他工具进行同步。如果机器间有公共挂载点，那么使用管道文件共享也是可以的。

启动一台worker，你需要设置Airflow并启动worker子命令：

```bash
airflow worker
```

一旦任务启动，从各自方向出发，你的worker就应该开始捡任务。

注意你也可以运行“Celery Flower”，用这个构建在Celery之上的web界面监控所有worker。你可以使用快速命令`airflow flower`来启动Flower web服务。

一些注意事项：

* 确保使用一个数据库支持的结果后端
* 确保在\[celery\_broker\_transport\_options\]中设置了可见性超时，这个时间应查过最长运行任务的估计到达时（ETA）
* 任务会消耗资源。确保你的worker有足够的资源运行_worker\_concurrency_个任务



