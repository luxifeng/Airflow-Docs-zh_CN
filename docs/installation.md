# 安装

## 获取Airflow

安装最新稳定版Airflow的最简单的方法是使用`pip`：

```python
pip install apache-airflow
```

你也可以安装支持更多特性（比如`s3`或`postgres`）的Airflow：

```python
pip install apache-airflow[postgres,s3]
```

{% hint style="info" %}
GPL依赖

Apache Airflow默认获取的版本会依赖GPL库（‘unidecode’）。如果这是个问题的化可以用`export SLUGIFY_USES_TEXT_UNIDECODE=yes`强制不依赖GPL库，然后继续正常安装。注意每次升级都要这样指定。同时也注意如果_unidecode_已经在系统中了，那么依赖还是会生效。
{% endhint %}

## 额外的包

`apache-airflow`这个基础PyPI包只会安装启动所需的东西。你可以根据你的运行环境安装可用的子包（subpackage）。比如，如果你不需要与Postgres连接，你就不用经历安装`postgres-devel`yum包的糟心了，同理适用于你正在使用的发行版本。

在后台，Airflow会有条件地导入依赖额外的包的操作符。

如下是子包列表及它们可以做的事情：

| subpackage | install command | enables |
| :--- | :--- | :--- |
| all | `pip install apache-airflow[all]` | All Airflow features known to man |
| all\_dbs | `pip install apache-airflow[all_dbs]` | All databases integrations |
| async | `pip install apache-airflow[async]` | Async worker classes for Gunicorn |
| celery | `pip install apache-airflow[celery]` | CeleryExecutor |
| cloudant | `pip install apache-airflow[cloudant]` | Cloudant hook |
| crypto | `pip install apache-airflow[crypto]` | Encrypt connection passwords in metadata db |
| devel | `pip install apache-airflow[devel]` | Minimum dev tools requirements |
| devel\_hadoop | `pip install apache-airflow[devel_hadoop]` | Airflow + dependencies on the Hadoop stack |
| druid | `pip install apache-airflow[druid]` | Druid related operators & hooks |
| gcp\_api | `pip install apache-airflow[gcp_api]` | Google Cloud Platform hooks and operators \(using `google-api-python-client`\) |
| hdfs | `pip install apache-airflow[hdfs]` | HDFS hooks and operators |
| hive | `pip install apache-airflow[hive]` | All Hive related operators |
| jdbc | `pip install apache-airflow[jdbc]` | JDBC hooks and operators |
| kerbero s | `pip install apache-airflow[kerberos]` | Kerberos integration for Kerberized Hadoop |
| ldap | `pip install apache-airflow[ldap]` | LDAP authentication for users |
| mssql | `pip install apache-airflow[mssql]` | Microsoft SQL Server operators and hook, support as an Airflow backend |
| mysql | `pip install apache-airflow[mysql]` | MySQL operators and hook, support as an Airflow backend. The version of MySQL server has to be 5.6.4+. The exact version upper bound depends on version of `mysqlclient` package. For example, `mysqlclient` 1.3.12 can only be used with MySQL server 5.6.4 through 5.7. |
| password | `pip install apache-airflow[password]` | Password authentication for users |
| postgres | `pip install apache-airflow[postgres]` | PostgreSQL operators and hook, support as an Airflow backend |
| qds | `pip install apache-airflow[qds]` | Enable QDS \(Qubole Data Service\) support |
| rabbitmq | `pip install apache-airflow[rabbitmq]` | RabbitMQ support as a Celery backend |
| redis | `pip install apache-airflow[redis]` | Redis hooks and sensors |
| s3 | `pip install apache-airflow[s3]` | `S3KeySensor`, `S3PrefixSensor` |
| samba | `pip install apache-airflow[samba]` | `Hive2SambaOperator` |
| slack | `pip install apache-airflow[slack]` | `SlackAPIPostOperator` |
| ssh | `pip install apache-airflow[ssh]` | SSH hooks and Operator |
| vertica | `pip install apache-airflow[vertica]` | Vertica hook support as an Airflow backend |

## 初始化Airflow数据库

Airflow要求你在能运行任务之前先初始化数据库。如果你仅仅是为了实验和学习Airflow，那么你可以继续使用默认的SQLite。如果你不想使用SQLite，那么可以看看[初始化数据库后端](how-to-guides.md#chu-shi-hua-shu-ju-ku-hou-duan)设置一个不同的数据库。

配置好数据库之后，跑任务之前，你需要初始化数据库：

```text
airflow initdb
```



