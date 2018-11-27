# 初始化数据库后端

如果你想对Airflow做真实的测试，你应考虑设置真正的数据库后端并且使用LocalExecutor。

由于Airflow使用强大的SqlAlchemy库与元数据进行交互，你可以使用任何数据库后端，只要它被SqlAlchemy后端所支持。我们推荐使用**MySQL**或**Postgres**。

{% hint style="info" %}
对于MySQL，为了获得合理的默认值，我们依赖更严格的ANSI SQL设置。请确保你的my.cnf文件中\[mysqld\]下已指明 _explicit\_defaults\_for\_timestamp=1_
{% endhint %}

{% hint style="info" %}
如果你决定使用**Postgres**，我们推荐使用`psycopg2`驱动，并且在你的SqlAlchemy连接语句中指定。同时也要注意，由于SqlAlchemy没有提供任何方式可以在Postgres连接URI中定位特定模式，你可能想要为你的角色设置默认模式，可以使用类似的命令 `ALTER ROLE username SET search_path = airflow, foobar;`
{% endhint %}

一旦你设置好了数据库来承载Airflow，你需要修改位于配置文件 `$AIRFLOW_HOME/airflow.cfg`的SqlAlchemy连接语句。随后你也应修改“executor”设置为“LocalExecutor”，这是一个能够在本地并行化运行实例的执行器。

```bash
# initialize the database
airflow initdb
```

