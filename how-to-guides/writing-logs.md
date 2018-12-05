# 写日志

### 写日志到本地

用户可以在`airflow.cfg`的`base_log_folder`一项中指定日志文件夹。默认是在`AIRFLOW_HOME`目录。

此外，用户可以提供远程地址，用于云端存储日志和备份日志。

在Airflow Web界面，本地日志优先于远程日志展示。如果本地日志找不到或无法获取，那么会展示远程日志。注意，日志仅在任务实例运行完毕（包括失败）后才会发送至远程存储。也就是说，正在运行的任务无法获取远程日志。日志在日志文件夹的存储路径为`{dag_id}/{task_id}/{execution_date}/{try_number}.log`。

### 写日志到Amazon S3

#### 开始之前

远程日志使用已存在的Airflow连接来读取/写入日志。如果你没有正确地配置连接，则会导致远程日志失败。

#### 启用远程日志

为启动这一特性，`airflow.cfg`必须如下文的例子配置：

```bash
[core]
# Airflow can store logs remotely in AWS S3. Users must supply a remote
# location URL (starting with either 's3://...') and an Airflow connection
# id that provides access to the storage location.
remote_base_log_folder = s3://my-bucket/path/to/logs
remote_log_conn_id = MyS3Conn
# Use server-side encryption for logs stored in S3
encrypt_s3_logs = False
```

在上述例子中，Airflow会尝试使用`S3Hook('MyS3Conn')`。

### 写日志到Azure Blob Storage

通过配置，Airflow可以读取和写入日志到Azure Blob Storage。按照如下步骤启用Azure Blob Storage的日志记录。

1. Airflow的日志系统要求有一份自定义.py文件放在`PYTHONPATH`，以便它可以从Airflow导入。从创建配置文件目录开始吧，推荐目录为`$AIRFLOW_HOME/config`。
2. 创建名为`$AIRFLOW_HOME/config/log_config.py`和`$AIRFLOW_HOME/config/__init__.py`的空文件；
3. 复制`airflow/config_templates/airflow_local_settings.py`的内容到上一步创建的`log_config.py`文件中；
4. 自行修改模板的下面部分：



   ```bash
   # wasb buckets should start with "wasb" just to help Airflow select correct handler
   REMOTE_BASE_LOG_FOLDER = 'wasb-<whatever you want here>'

   # Rename DEFAULT_LOGGING_CONFIG to LOGGING CONFIG
   LOGGING_CONFIG = ...
   ```

5. 确保Azure Blob Storage（Wasb）的连接hook已经定义。hook应能读取和写入Azure Blob Storage bucket，如上文`REMOTE_BASE_LOG_FOLDER`所定义；
6. 更新`$AIRFLOW_HOME/airflow.cfg`，使其包含：



   ```bash
   remote_logging = True
   logging_config_class = log_config.LOGGING_CONFIG
   remote_log_conn_id = <name of the Azure Blob Storage connection>
   ```

7. 重启Airflow webserver和scheduler，触发（或等待）新任务执行；
8. 验证新执行任务的日志出现在你所定义的bucket中。

### 写日志到Google Cloud Storage

按照以下步骤启用Google Cloud Storage日志记录。

1. Airflow的日志系统要求有一份自定义.py文件放在`PYTHONPATH`，以便它可以从Airflow导入。从创建配置文件目录开始吧，推荐目录为`$AIRFLOW_HOME/config`。
2. 创建名为`$AIRFLOW_HOME/config/log_config.py`和`$AIRFLOW_HOME/config/__init__.py`的空文件；
3. 复制`airflow/config_templates/airflow_local_settings.py`的内容到上一步创建的`log_config.py`文件中；
4. 自行修改模板的下面部分：



   ```bash
   # Add this variable to the top of the file. Note the trailing slash.
   GCS_LOG_FOLDER = 'gs://<bucket where logs should be persisted>/'

   # Rename DEFAULT_LOGGING_CONFIG to LOGGING CONFIG
   LOGGING_CONFIG = ...

   # Add a GCSTaskHandler to the 'handlers' block of the LOGGING_CONFIG variable
   'gcs.task': {
       'class': 'airflow.utils.log.gcs_task_handler.GCSTaskHandler',
       'formatter': 'airflow.task',
       'base_log_folder': os.path.expanduser(BASE_LOG_FOLDER),
       'gcs_log_folder': GCS_LOG_FOLDER,
       'filename_template': FILENAME_TEMPLATE,
   },

   # Update the airflow.task and airflow.task_runner blocks to be 'gcs.task' instead of 'file.task'.
   'loggers': {
       'airflow.task': {
           'handlers': ['gcs.task'],
           ...
       },
       'airflow.task_runner': {
           'handlers': ['gcs.task'],
           ...
       },
       'airflow': {
           'handlers': ['console'],
           ...
       },
   }
   ```

5. 确保Google Cloud Platform的连接hook已经定义。hook应能读取和写入Google Cloud Storage bucket，如上文`GCS_LOG_FOLDER`所定义；
6. 更新`$AIRFLOW_HOME/airflow.cfg`，使其包含：
7. 重启Airflow webserver和scheduler，触发（或等待）新任务执行；



   ```bash
   task_log_reader = gcs.task
   logging_config_class = log_config.LOGGING_CONFIG
   remote_log_conn_id = <name of the Google cloud platform hook>
   ```

8. 验证新执行任务的日志出现在你所定义的bucket中；
9. 验证Google Cloud Storage视图运行在界面上。拉取新执行的任务，检验你能看到类似输出：



   ```bash
   *** Reading remote log from gs://<bucket where logs should be persisted>/example_bash_operator/run_this_last/2017-10-03T00:00:00/16.log.
   [2017-10-03 21:57:50,056] {cli.py:377} INFO - Running on host chrisr-00532
   [2017-10-03 21:57:50,093] {base_task_runner.py:115} INFO - Running: ['bash', '-c', u'airflow run example_bash_operator run_this_last 2017-10-03T00:00:00 --job_id 47 --raw -sd DAGS_FOLDER/example_dags/example_bash_operator.py']
   [2017-10-03 21:57:51,264] {base_task_runner.py:98} INFO - Subtask: [2017-10-03 21:57:51,263] {__init__.py:45} INFO - Using executor SequentialExecutor
   [2017-10-03 21:57:51,306] {base_task_runner.py:98} INFO - Subtask: [2017-10-03 21:57:51,306] {models.py:186} INFO - Filling up the DagBag from /airflow/dags/example_dags/example_bash_operator.py
   ```

   注意首行文字表示它正在读取远程日志文件。

请注意，如果你曾经使用旧式airflow.cfg的配置方法将日志保存到Google Cloud Storage，那么旧日志不会显示在Airflow界面上，虽然它们仍然存在在Google Cloud Storage上。这是一个向后不兼容的变化。如果你对此不满意，你可以改变`FILENAME_TEMPLATE`来表明旧式日志文件格式。

