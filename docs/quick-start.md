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

```bash
# run your first task instance
airflow run example_bash_operator runme_0 2015-01-01
# run a backfill over 2 days
airflow backfill example_bash_operator -s 2015-01-01 -e 2015-01-02
```

## 下一步是什么？

从现在开始，你可以打开[教程](tutorial.md)章节了解更多的示例，或者你准备好了小试牛刀，那么就去[指南](how-to-guides.md)章节看一看吧。

