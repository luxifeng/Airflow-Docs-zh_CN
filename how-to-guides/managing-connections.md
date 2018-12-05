# 管理连接

Airflow需要知道如何连接到你的环境。其他系统和服务的主机名、端口、登录账户和密码等等信息，都在用户界面的`Admin->Connection`部分处理。你编写的pipeline代码可以引用连接对象的'conn\_id'。

![](../.gitbook/assets/connections.png)

可以通过用户界面或者环境变量创建和管理连接。

更多信息请访问[Connenctions Concepts](https://airflow.apache.org/concepts.html#concepts-connections)文档。

### 使用用户界面创建连接

打开用户界面上的`Admin->Connection`部分。点击`Create`链接创建新的连接。

![](../.gitbook/assets/connection_create.png)

1. 在`Conn Id`字段填上想要的连接ID。推荐使用小写字母和下划线。
2. 在`Conn Type`字段选择连接类型。
3. 填写剩下的字段。访问[Connection Types](https://airflow.apache.org/howto/manage-connections.html#manage-connections-connection-types)查看不同连接类型下的字段描述。
4. 点击`Save`按钮创建连接。

### 使用用户界面编辑连接

打开用户界面上的`Admin->Connection`部分。点击连接列表中你想要编辑的连接旁边的铅笔图标。

![](../.gitbook/assets/connection_edit.png)

修改连接属性，点击`Save`按钮保存修改。

### 使用环境变量创建连接

Airflow pipeline中的连接可以使用环境变量创建。为正确使用连接，环境变量需为Airflow设置前缀为`AIRFLOW_CONN_`，值的格式为URI格式。

引用Airflow pipeline中的连接时，参数`conn_id`应为无前缀的变量名。例如，如果`conn_id`命名为`postgres_master`，那么相应的环境变量就应命名为`AIRFLOW_CONN_POSTGRES_MASTER`（注意环境变量都为大写字母）。Airflow假设环境变量返回的值是URI格式（如：`postgres://user:password@localhost:5432/master`或`s3://accesskey:secretkey@S3`）。

### 连接类型

### Google Cloud Platform

Google Cloud Platform连接类型允许GCP集成。

#### GCP身份验证

有两种使用Airflow连接GCP的方式。

1. 使用[应用默认凭证](https://google-auth.readthedocs.io/en/latest/reference/google.auth.html#google.auth.default)，例如通过运行在Google Compute Engine上的元数据服务器。
2. 使用硬盘上的[服务账户](https://cloud.google.com/docs/authentication/#service_accounts)密钥文件（JSON格式）

#### 默认连接ID

默认连接ID如下：

`bigquery_default`：为[`BigQueryHook`](https://airflow.apache.org/integration.html#airflow.contrib.hooks.bigquery_hook.BigQueryHook)钩子所用。

`google_cloud_datastore_default`：为[`DatastoreHook`](https://airflow.apache.org/integration.html#airflow.contrib.hooks.datastore_hook.DatastoreHook)钩子所用。

`google_cloud_default`：为[`GoogleCloudBaseHook`](https://airflow.apache.org/code.html#airflow.contrib.hooks.gcp_api_base_hook.GoogleCloudBaseHook)、[`DataFlowHook`](https://airflow.apache.org/integration.html#airflow.contrib.hooks.gcp_dataflow_hook.DataFlowHook)、[`DataProcHook`](https://airflow.apache.org/code.html#airflow.contrib.hooks.gcp_dataproc_hook.DataProcHook)、[`MLEngineHook`](https://airflow.apache.org/integration.html#airflow.contrib.hooks.gcp_mlengine_hook.MLEngineHook)和[`GoogleCloudStorageHook`](https://airflow.apache.org/integration.html#airflow.contrib.hooks.gcs_hook.GoogleCloudStorageHook)钩子所用。

#### 配置连接

**Project Id（必需）**

        要连接到的Google Cloud项目ID。

**Keyfile Path**

        硬盘上的[服务账号](https://cloud.google.com/docs/authentication/#service_accounts)密钥文件（JSON格式）路径。

        如果使用应用默认凭证，该项非必需。

**Keyfile JSON**

         硬盘上的[服务账号](https://cloud.google.com/docs/authentication/#service_accounts)密钥文件的内容。使用这种身份认证方式时推荐[保护你的连接](https://airflow.apache.org/howto/secure-connections.html)。

        如果使用应用默认凭证，该项非必需。

**Scopes（半角逗号分隔）** 

        需身份验证的Google Cloud scopes，是用半角逗号分隔的列表。

{% hint style="info" %}
如果使用应用默认凭证，scope会被忽略。见问题 [AIRFLOW-2522](https://issues.apache.org/jira/browse/AIRFLOW-2522)。
{% endhint %}



