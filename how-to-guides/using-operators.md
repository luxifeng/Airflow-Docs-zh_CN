# 使用operator

一个operator代表一个单独的、理想情况下是幂等的任务。Operator决定了当你的DAG在运行时实际在执行什么。

更多信息请访问[Operator Concepts](https://airflow.apache.org/concepts.html#concepts-operators)文档和[Operators API Reference](https://airflow.apache.org/code.html#api-reference-operators)。

## BashOperator

使用 [`BashOperator`](https://airflow.apache.org/code.html#airflow.operators.bash_operator.BashOperator)来执行[Bash](https://www.gnu.org/software/bash/) shell命令。

```python
run_this = BashOperator(
    task_id='run_after_loop', bash_command='echo 1', dag=dag)
```

### 制作模板

你可以使用Jinja模板来参数化`bash_command`的参数。

```python
task = BashOperator(
    task_id='also_run_this',
    bash_command='echo "run_id={{ run_id }} | dag_run={{ dag_run }}"',
    dag=dag)
```

### 疑难解答

#### Jinja模板找不到

用`bash_command`参数直接调用Bash脚本时要在脚本名字后面添加一个空格。因为Airflow会尝试将一个Jinja模板应用于此，但行不通。

```python
t2 = BashOperator(
    task_id='bash_example',

    # This fails with `Jinja template not found` error
    # bash_command="/home/batcher/test.sh",

    # This works (has a space after)
    bash_command="/home/batcher/test.sh ",
    dag=dag)
```

## PythonOperator

使用[`PythonOperator`](https://airflow.apache.org/code.html#airflow.operators.python_operator.PythonOperator)来执行Python可调用函数。

```python
def print_context(ds, **kwargs):
    pprint(kwargs)
    print(ds)
    return 'Whatever you return gets printed in the logs'


run_this = PythonOperator(
    task_id='print_the_context',
    provide_context=True,
    python_callable=print_context,
    dag=dag)
```

### 传入参数

使用`op_args`和`op_kwargs`参数来传递额外的参数给Python可调用函数。

```python
def my_sleeping_function(random_base):
    """This is a function that will run within the DAG execution"""
    time.sleep(random_base)


# Generate 10 sleeping tasks, sleeping from 0 to 4 seconds respectively
for i in range(5):
    task = PythonOperator(
        task_id='sleep_for_' + str(i),
        python_callable=my_sleeping_function,
        op_kwargs={'random_base': float(i) / 10},
        dag=dag)

    task.set_upstream(run_this)
```

### 制作模板

当你设置`provide_context`参数为`True`时，Airflow会传入其他关键词参数集：每一个都对应着一个Jinja模板变量和一个`templates_dict`参数。

`templates_dict`参数已经模板化了，因此字典中的每一个值都会被判断为Jinja模板。

## Google Cloud Platform Operators

### GoogleCloudStorageToBigQueryOperator

使用[`GoogleCloudStorageToBigQueryOperator`](https://airflow.apache.org/integration.html#airflow.contrib.operators.gcs_to_bq.GoogleCloudStorageToBigQueryOperator)来执行BigQuery加载作业。

```python
load_csv = gcs_to_bq.GoogleCloudStorageToBigQueryOperator(
    task_id='gcs_to_bq_example',
    bucket='cloud-samples-data',
    source_objects=['bigquery/us-states/us-states.csv'],
    destination_project_dataset_table='airflow_test.gcs_to_bq_table',
    schema_fields=[
        {'name': 'name', 'type': 'STRING', 'mode': 'NULLABLE'},
        {'name': 'post_abbr', 'type': 'STRING', 'mode': 'NULLABLE'},
    ],
    write_disposition='WRITE_TRUNCATE',
    dag=dag)
```

