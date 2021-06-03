# apache-airflow-guide

# Install and Setup

**Install**

```
pip3 install apache-airflow
```

# Essential Concepts

## Graphs and DAGs

<img width="567" alt="Screen Shot 2021-06-03 at 0 37 49" src="https://user-images.githubusercontent.com/51218415/120592717-ef14dd00-c403-11eb-965d-ccad00af65d4.png">

### Graphs Intro
- Node or Vertices Connected by Edges
- Different types of Graphs based on different restrictions like Trees, Cyclic, Acyclic, Directed, Undirected and Weighted
- Examples include rail networks, social networks, the web!
- We are not referring to bar graphs

### DAG
- Each Workflow in Airflow can be represented as a DAG
- Directed -- all the edges bear a direction (think on way traffic)
- Acyclic -- so it has no cycles (think of a tree like strcuture)
- Graph -- a set of nodes and edges
 

<img width="705" alt="Screen Shot 2021-06-03 at 1 32 35" src="https://user-images.githubusercontent.com/51218415/120598183-947f7f00-c40b-11eb-8913-1f98a20f14f8.png">

<img width="748" alt="Screen Shot 2021-06-03 at 1 33 05" src="https://user-images.githubusercontent.com/51218415/120598247-a7924f00-c40b-11eb-8dae-7bf9f1399a95.png">

# Hello AirFlow!

**HelloWorld.py**

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
#slack
from airflow.hooks.base_hook import BaseHook
from airflow.contrib.operators.slack_webhook_operator import SlackWebhookOperator

from datetime import datetime, timedelta
from airflow.models import Variable

#XCOM abbreviation for cross communication
#push and pull


default_args = {
    'owner': 'Vaga',
    'depends_on_past': False,
    'start_date': datetime(2018, 10, 1),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 2,
    'retry_delay': timedelta(minutes=2),
    'provide_context': True,
}
dag = DAG(
    'Hello', default_args=default_args, schedule_interval="0 * * * *")

Stage1 = BashOperator(
    task_id='Hello',
    bash_command='echo {{ var.value.Jordan}}',
    dag=dag)

Stage2 = BashOperator(
    task_id='World',
    bash_command='echo world',
    dag=dag)
#function to get the number 7
def seven():
    return 7
#First Way to push using xcom
Stage3 = PythonOperator(
     task_id = 'try_xcom7',
     python_callable = seven,
     xcom_push=True,
     dag = dag)

def pushnine(**context):
    context['ti'].xcom_push(key='keyNINE', value=9)

#second way to push
Stage5 = PythonOperator(
    task_id = 'push9',
    python_callable = pushnine,
    dag = dag
    )

def getNINE(**context):
    value = context['ti'].xcom_pull(key='keyNINE',task_ids='push9')
    print (value)
    return value

#Pull values 
Stage4 = PythonOperator(
    task_id ='pull_xcom9',
    python_callable=getNINE,
    provide_context=True,
    dag=dag
    )

def tell_slack(**context):
    webhook = BaseHook.get_connection('Slack2').password
    message = "hey there! we connected to slack"
    alterHook = SlackWebhookOperator(
        task_id = 'integrate_slack',
        http_conn_id='Slack2',
        webhook_token=webhook,
        message = message,
        username='Vaga',
        dag=dag)
    return alterHook.execute(context=context)


Stage7 = PythonOperator(
     task_id ='slack_task2',
     python_callable=tell_slack,
     provide_context=True,
     dag=dag
   )


Stage1 >> Stage2
# Stage2.set_downstream(Stage1)
# Stage1.set_upstream(Stage2)

# Stage1 << Stage2
# Stage1.set_downstream(Stage2)
# Stage2.set_upstream(Stage1)
```

```bash
python3 HelloWorld.py
```

```bash
airflow list_dags
```

```bash
airflow list_dags Hello --tree
```

```bash
airflow test Hello Hello 2021
```

```bash
airflow test Hello World 2021
```

```bash
airflow backfill Hello -s 2021-01-01 -e 2021-01-02
```

In other terminal:

```bash
airflow webserver
```

Open your browser at:
http://localhost:8080

Browse > DAG runs

<img width="1275" alt="Screen Shot 2021-06-03 at 2 21 18" src="https://user-images.githubusercontent.com/51218415/120604172-63ef1380-c412-11eb-9069-d1a6bfab8312.png">

# AirFlow UI

```bash
airflow webserver -p 8080
```

**Restart**

```bash
lsof -i tcp:8080
```

```bash
kill <YOUR_PID>
```

```bash
airflow webserver
```

<img width="1410" alt="Screen Shot 2021-06-03 at 2 33 19" src="https://user-images.githubusercontent.com/51218415/120606408-c812d700-c414-11eb-940e-124d778f46cc.png">
<img width="1408" alt="Screen Shot 2021-06-03 at 2 33 30" src="https://user-images.githubusercontent.com/51218415/120606420-cba65e00-c414-11eb-93ed-08b9a6b1b1ea.png">
<img width="1409" alt="Screen Shot 2021-06-03 at 2 33 38" src="https://user-images.githubusercontent.com/51218415/120606423-cc3ef480-c414-11eb-8f3f-39ae85b24a09.png">
<img width="1409" alt="Screen Shot 2021-06-03 at 2 33 43" src="https://user-images.githubusercontent.com/51218415/120606429-ccd78b00-c414-11eb-9ba4-dd5b6d139ecc.png">
<img width="1410" alt="Screen Shot 2021-06-03 at 2 33 51" src="https://user-images.githubusercontent.com/51218415/120606432-cd702180-c414-11eb-80e5-6d7de86103a0.png">
<img width="1410" alt="Screen Shot 2021-06-03 at 2 34 14" src="https://user-images.githubusercontent.com/51218415/120606433-ce08b800-c414-11eb-894c-c44d94bc1448.png">
<img width="1386" alt="Screen Shot 2021-06-03 at 2 34 34" src="https://user-images.githubusercontent.com/51218415/120606435-ce08b800-c414-11eb-9123-4d5939da6067.png">
<img width="1403" alt="Screen Shot 2021-06-03 at 2 34 45" src="https://user-images.githubusercontent.com/51218415/120606439-cea14e80-c414-11eb-9181-4eb80978668c.png">
<img width="1404" alt="Screen Shot 2021-06-03 at 2 35 37" src="https://user-images.githubusercontent.com/51218415/120606440-cea14e80-c414-11eb-8aaa-3b2d444a4251.png">
<img width="1402" alt="Screen Shot 2021-06-03 at 2 36 12" src="https://user-images.githubusercontent.com/51218415/120606441-cea14e80-c414-11eb-8a88-6dfbdaa6c0e6.png">
<img width="1395" alt="Screen Shot 2021-06-03 at 2 36 35" src="https://user-images.githubusercontent.com/51218415/120606446-cf39e500-c414-11eb-8c9e-229f0d6705e2.png">
<img width="1409" alt="Screen Shot 2021-06-03 at 2 36 58" src="https://user-images.githubusercontent.com/51218415/120606447-cfd27b80-c414-11eb-990b-2f54a9a2262b.png">
<img width="1396" alt="Screen Shot 2021-06-03 at 2 37 05" src="https://user-images.githubusercontent.com/51218415/120606450-cfd27b80-c414-11eb-9391-c07b7d59f1d7.png">
<img width="1397" alt="Screen Shot 2021-06-03 at 2 37 12" src="https://user-images.githubusercontent.com/51218415/120606452-d06b1200-c414-11eb-91a8-9644c2f79c28.png">





