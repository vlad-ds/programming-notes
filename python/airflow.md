# Airflow

#### Definitions

*Data Engineering.* Taking any action involving data and turning it into a reliable, repeatable, and maintainable process.

*Workflow.* A set of steps to accomplish a given data engineering task. 

*Airflow.* A platform for creating, scheduling and monitoring workflows. Can implement programs from any language, but workflows are written in Python. Workflows are implemented as DAGs. 

Other tools: Luigi, SSIS, Bash scripting. 

*DAG*. Directed acyclic graph. Consists of tasks and the dependencies between tasks. 

#### Basic workflow

````python
from airflow.models import DAG
from datetime import datetime

default_arguments = {
    'owner': 'jdoe',
    'email': 'joe@datacamp.com',
    'start_date': datetime(2020, 1, 20)
}

etl_dag = DAG('etl_workflow', default_args=default_arguments)
````

Using the airflow command line tool: 

```bash
airflow list_dags
```

Use the command line to start Airflow processes, manually run DAGs and Tasks, and get logging information. Use Python to create a DAG and edit the individual properties of a DAG. 

To start the airflow webserver:

```
airflow webserver -p 9090
```

##### Operators

An operator represents a single task in a workflow.

Typically operators operate independently (all the resources to complete the task are available within the operator).

Generally do not share information.

Several types like `DummyOperator`, `BashOperator` and `PythonOperator`. 

Operators can be found in the `airflow.operators` library. 

```python
from airflow.operators.bash_operator import BashOperator
BashOperator(
		task_id='bash_example',
		bash_command='echo "Example!"',
		dag=etl_dag)
```

```python
from airflow.operators.python_operator import PythonOperator
def sleep(length_of_time):
    time.sleep(length_of_time)
python_task = PythonOperator(
	task_id='simple_print',
	python_callable=sleep,
    op_kwargs={'length_of_time': 5}
	dag=example_dag)
```

```python
from airflow.operators.email_operator import EmailOperator
email_task = EmailOperator(
	task_id='email_sales_report',
	to='sales_manager@example.com',
	subject='Automated Sales Report',
	html_content='Attached is the latest sales report',
	files='latest_sales.xlsx',
	dag=example_dag)
```

Operator gotchas:

* Not guaranteed to run in the same location/environment. 
* May require extensive use of environment variables (e.g. AWS credentials).
* Can be difficult to run tasks with elevated privileges. 

##### Tasks

They are instances of operators. Usually assigned to a Python variable. 

```python
example_task = BashOperator(task_id='bash_example',
                           bash_command='echo "Example!"',
                           dag=dag)
```

Task dependencies define a given order of task completion. Without dependencies, Airflow executes tasks without guarantees of order. 

Tasks can be *upstream* or *downstream*. An upstream task must complete prior to any downstream tasks. Upstream = Before, Downstream = After. 

Task dependencies are defined using the bitshift operators: `>>` (upstream) and `<<` (downstream). 

```python
task1 >> task2 #or task2 << task1
task1 >> task2 >> task3 >> task4 #chained dependencies
task1 >> task2 << task3 #mixed dependencies 
```
##### Scheduling

A DAG run is a specific instance of a workflow at a point in time. A DAG can be run manually or via the `schedule_interval` parameter. Each run maintains a state for itself and the tasks within. The DAGs can have a `running`, `failed` or `success` state. 

When scheduling a DAG:

`start_date` is when to initially schedule the run

`end_date` is when to stop running new instances (optional)

`max_tries` is how many attempts to make

`schedule_interval` how often to run between `start_date` and `end_date`. Can be defined via `cron` style syntax or via built-in presets like `@hourly`. `None` means never schedule DAG, `@once` means scheduling only once. 

When starting the DAG, Airflow will consider `start_date` as the earliest possible value, but won't actually execute the DAG until one schedule interval has passed beyond the start date. 

##### Sensors

Sensors can be found in `airflow.sensors` and `airflow.contrib.sensors`. 

An operator that waits for a condition to be true. E.g. creation of a file, upload of a database record, a certain response from a web request. You can define how often to check for a condition to be true. 

`mode` can be `'poke'`  (runs repeatedly) or `'reschedule'` (gives up task slot and tries again later). 

`poke_interval` defines how often to check for condition. 

`timeout ` how many seconds to wait before failing task. 

```python
from airflow.contrib.sensors.file_sensor import FileSensor

file_sensor_task = FileSensor(task_id='file_sense',
                             filepath='salesdata.csv',
                             poke_interval=300,
                             dag=sales_report_dag)

init_sales_cleanup >> file_sensor_task >> generate_report
```

When to use a sensor:

* You're uncertain when a condition will be true
* If failure not immediately desired
* Add task repetition without loops

#### Executors

They run tasks. 

`SequentialExecutor`is functional for learning and testing, but not recommended for production. 

`LocalExecutor` runs on a single system. Treats each task as a process. Parallelism defined by the user. 

`CeleryExecutor` uses a Celery backend as task manager. 

You can determine the executor by looking at `airflow.cfg`. 

```
cat airflow/airflow.cfg | grep "executor = "
```

You can also run `airflow list_dags`. 

#### SLA

Service Level Agreement. Indicates the amount of time a task or a DAG should require to run. An *SLA miss* is any time the task/DAG does not meet the expected timing. In that case an email alert is sent and a note is made in the log. 

SLAs can be defined by passing a `sla` argument to the task creation with a `timedelta`. Or you can define an `'sla'` key in the `default_args` dictionary. This is passed to the DAG instance and will apply to any tasks internally. 

Airflow provides arguments for setting up email alerts. 

#### Templates

Allow substitution of information during a DAG run. Add flexibility to tasks. 

They are created using the `Jinja` templating language. 

```python
templated_command = """
	echo "Reading {{params.filename}}"
"""

t1 = BashOperator(task_id='template_task',
                 bash_command=templated_command,
                 params={'filename': 'file1.txt'},
                 dag=example_dag)
```

```python
templated_command="""
{% for filename in params.filenames %}
	echo "Reading {{ filename }}"
{% endfor %}
"""
t1 = BashOperator(task_id='template_task',
                 bash_command=templated_command,
                 params={'filenames': ['file1.txt', 'file2.txt']},
                 dag=example_dag)
```

Airflow provides a number of built-in runtime variables. E.g. `{{ ds }}` is the execution date. 

#### Branching

Provides conditional logic within airflow. Tasks can be executed or skipped depending on the result of an operator. By default we use a `BranchPythonOperator`. Takes a `python_callable` to return the next task id (or list of ids) to follow. 

```python
def branch_test(**kwargs):
    if int(kwargs['ds_nodash']) % 2 == 0:
        return 'even_day_task'
    else:
        return 'odd_day_task'
    
branch_task = BanchPythonOperator(task_id='branch_task',
                                 dag=dag,
                                 provide_context=True,
                                 python_callable=branch_test)
```

### Production pipelines

To run a specific task:

```
airflow run <dag_id> <task_id> <date>
```

To run a full DAG:

```
airflow trigger_dag -e <date> <dag_id>
```

---

A concise and elegant DAG definition: 

```python
spark_args = {"py_files": dependency_path,
              "conn_id": "spark_default"}

# Define ingest, clean and transform job.
with dag:
    ingest = BashOperator(task_id='Ingest_data', 
                          bash_command='tap-marketing-api | target-csv --config %s' % config)
    clean = SparkSubmitOperator(application=clean_path, 
                                task_id='clean_data', 
                                **spark_args)
    insight = SparkSubmitOperator(application=transform_path, 
                                  task_id='show_report', 
                                  **spark_args)
    
    # set triggering sequence
    ingest >> clean >> insight
```

---

## Deploying Airflow

In a clean Linux image:

```bash
export AIRFLOW_HOME=~/airflow
pip install apache-airflow
airflow initdb #creates the airflow/ directory
```

Default Airflow DB is SQLite. The `.cfg` file defines which Executor will be used. Default is `SequentialExecutor`. 

The `tests` folder will have unit tests for deployment, and ensure consistency across DAGs. 

To upload your DAGs to the server, you can clone your repo on the Airflow server. Or you can copy the DAG file to the server with a tool like `rsync`. Or you make use of packaged DAGs, which are zipped archives that promote isolation between projects. 

