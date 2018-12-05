# 使用systemd运行Airflow

Airflow能够与基于systemd的系统集成。这使得你的守护进程易于监控，因为systemd会注意在失败时重启守护进程。在`scripts/systemd`目录你可以发现一些unit文件，这些文件已经在基于Redhat的系统上测试过。你可以复制这些文件到`/usr/lib/systemd/system`。假定Airflow会运行在`airflow:airflow`下。如果没有运行（或者是运行在非基于Redhat的系统上）你可能需要调整这些unit文件。

环境配置取自于`/etc/sysconfig/airflow`。已提供一个实例文件供参考。运行调度程序的时候要确保已在该文件中指明了`SCHEDULER_RUNS`变量。你也可以在这里定义，例如，`AIRFLOW_HOME`或`AIRFLOW_CONFIG`。

