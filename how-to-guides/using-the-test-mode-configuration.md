# 使用测试模式配置

Airflow有一套固定的“测试模式”配置选项。你可以通过调用`airflow.configuration.load_test_config()`在任意时候加载它们（注意这些操作是不可逆的！）但是，一些选项（比如DAG\_FOLDER）在你有机会调用load\_test\_config\(\)之前就已经加载了。为了尽快加载测试配置，在airflow.cfg中设置test\_mode：

```bash
[tests]
unit_test_mode = True
```

由于Airflow的自动环境变量扩展（见[设置配置选项](https://airflow.apache.org/howto/set-config.html)），你也可以设置环境变量`AIRFLOW__CORE__UNIT_TEST_MODE`来暂时性地覆盖airflow.cfg中的配置。

