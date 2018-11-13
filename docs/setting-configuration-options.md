# 设置配置项

第一次运行Airflow时，Airflow会创建一个文件，叫做`airflow.cfg`，放在`$AIRFLOW_HOME`目录下（默认是`~/airflow`）。这个文件包含了Airflow的各种配置，你可以通过编辑它来改变任何设置。你也可以通过环境变量来设置选项，格式是： `$AIRFLOW__{SECTION}__{KEY}`（注意双下划线）。

例如，元数据库连接语句可以在`airflow.cfg`设置，如下：

```text
[core]
sql_alchemy_conn = my_conn_string
```

或者创建相应的环境变量：

```text
AIRFLOW__CORE__SQL_ALCHEMY_CONN=my_conn_string
```

你也可以在配置项键后面添加`_cmd`，在运行时派生连接语句：

```text
[core]
sql_alchemy_conn_cmd = bash_command_to_run
```

——但是只有三种配置像能够从命令获取：sql\_alchemy\_conn，broker\_url和result\_backend。其背后的逻辑是不在服务器上的纯文本文件中保存密码。配置生效的优先级顺序如下：

1. 环境变量
2. airflow.cfg中的配置
3. airflow.cfg中的命令
4. 默认配置

