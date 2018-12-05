# 使用upstart运行Airflow

Airflow可以与基于upstart的系统集成。upstart会自动启动所有Airflow服务，只要系统启动时相应的`*.conf`文件存在在`/etc/init`下。一旦失败，upstart会自动重启所有的进程（直到达到`*.conf`文件所设置的 respawn limit）。

你可以在`scripts/upstart`文件夹下发现upstart作业示例文件。这些文件已经在Ubuntu 14.04 LTS系统上测试过。你可能不得不调整`start on`和`stop on`小节，使之能运行在其他upstart系统上。一些可能的选项已列在`scripts/upstart/README`。

按需修改`*.conf`文件并复制到`/etc/init`目录。假定Airflow会运行在`airflow:airflow`下。如果你要使用其他user/group，修改`*.conf`文件中的`setuid`和`setgid`。

你可以使用`initctl`来手动启动、停止、观察与upstart集成的Airflow进程状态

```bash
initctl airflow-webserver status
```

