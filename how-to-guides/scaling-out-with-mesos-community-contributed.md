# 使用Mesos横向扩展（社区贡献）

有两种方式可以将Airflow作为mesos framework进行运行。

1. 直接在mesos从节点运行Airflow任务，这要求每个mesos从节点都已安装和配置AIrflow；
2. 在已安装Airflow的docker容器里运行Airflow任务，docker容器运行在一个mesos从节点上。

### 任务直接在mesos从节点执行

`MesosExecutor`允许你在一个Mesos集群上调度Airflow。为使其可行，你需要一个运行中的mesos集群，且必须如下操作：

1. 在一个mesos从节点上安装Airflow，该节点是web服务和调度程序将要运行的节点，让我们称之为“AIrflow服务器”；
2. 在Airflow服务器上，安装mesos python eggs，可从 [mesos downloads](http://open.mesosphere.com/downloads/mesos/)下载；
3. 在Airflow服务器上，使用能被所有的mesos从节点访问的数据库（比如MySQL），并在添加`airflow.cfg`配置；
4. 修改`airflow.cfg`，将executor参数改为_MesosExecutor_，并提供相关的Mesos配置；
5. 在所有的mesos从节点上，安装Airflow。从Airflow服务器复制`airflow.cfg`（以便使用相同的sql alchemy连接）；
6. 在所有的mesos从节点上，为服务日志运行如下命令：



   ```text
   airflow serve_logs
   ```

7. 在Airflow服务器上，要启动处理/调度mesos上的DAG，运行：



   ```bash
   airflow scheduler -p
   ```

注意：我们需要-p参数来腌制DAG。（？）

现在你可以看到Airflow framework和相应的任务出现在mesos界面。Airflow任务日志可以入场在Airflow界面看到。

更多关于mesos的信息，请访问[mesos文档](http://mesos.apache.org/documentation/latest/)。任何关于_MesosExecutor_的询问或bug，请联系[@kapil-malik](https://github.com/kapil-malik)。

### 任务在mesos从节点上的容器中执行



