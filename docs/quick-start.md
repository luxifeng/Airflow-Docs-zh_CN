# 快速启动

Airflow的安装快速且简便。

```bash
# airflow needs a home, ~/airflow is the default,
# but you can lay foundation somewhere else if you prefer
# (optional)
export AIRFLOW_HOME=~/airflow

# install from pypi using pip
pip install apache-airflow

# initialize the database
airflow initdb

# start the web server, default port is 8080
airflow webserver -p 8080

# start the scheduler
airflow scheduler

# visit localhost:8080 in the browser and enable the example dag in the home page
```

运行以上命令，Airflow会创建`$AIRFLOW_HOME`文件夹，并且会放置一个带有默认配置的文件“airflow.cfg”，这个文件可以帮助你快速开始。你可以打开`$AIRFLOW_HOME/airflow.cfg`查看这个文件，或者通过UI的`Admin->Configuration`菜单进行查看。Web服务器的PID文件存储在 `$AIRFLOW_HOME/airflow-webserver.pid`，或者进程由systemd启动的话，PID文件存储在 `/run/airflow/webserver.pid`。

Airflow使用SQLite数据库，方便你开箱即用，不过由于使用SQLite数据库后端不能做到并行化，因此你很快就不会使用它了。使用SQLite要配合使用`SequentialExecutor`，这钟执行器只能支持顺序跑任务实例。虽然功能有限，但是借此你可以快速上手并且浏览一遍UI和命令行工具。

这里有一些命令可以触发一些任务实例。跑一跑下边的命令，你应该能在`example1`DAG图中看到这些作业（job）的状态改变了。

```bash
# run your first task instance
airflow run example_bash_operator runme_0 2015-01-01
# run a backfill over 2 days
airflow backfill example_bash_operator -s 2015-01-01 -e 2015-01-02
```

## 下一步做什么？

从现在开始，你可以打开[教程](tutorial.md)章节了解更多的示例，或者你准备好了小试牛刀，那么就去[指南](how-to-guides.md)章节看一看吧。

