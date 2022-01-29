---
layout: single
title:  "Airflow DAG 생성 / Operator 구성"
category: Airflow
tag: Airflow
---

> 오늘은 Airflow의 DAG Task를 생성하는 방법에 대해서 정리해보려 한다. 기본적으로 DAG을 어떻게 생성하는지와 Operator 의 종류 및 구성에 대해서 살펴보자.


<br>

# DAG 생성하기
DAG을 생성하는 위치는 `airflow init` 명령어를 수행하면 생기는 `airflow.cfg`에 기록되어 있다. `[core] - dags_folder` 의 경로가 dag이 위치해야 하는 경로이다.
해당 Directory에 python 파일을 생성하면 dag이 추가되는 형태이다. 생성하고 싶은 단어를 입력해서 dag file 하나를 만들어보자.

DAG은 아래와 같이 airflow Module에서 가져올 수 있다. 

## DAG 생성 예시
- airflow module에서 DAG 호출.
- datetime.datetime을 사용해 날짜를 다룰 수 있다.
- default_args : dict 자료형으로 기본 인자를 설정할 수 있다. DAG안에 있는 Operator에 설정할 공통 속성에 대해서 이곳에 정의한다. 그렇기에 BaseOperator에 있는 값을 default_args 에 전달할 수 있다.
  - 인자값 설명 : https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/models/index.html#airflow.models.BaseOperator

  ``` python
  from airflow import DAG
  from datetime import datetime, timedelta

  default_args = {
      'start_date' : datetime(2022, 1, 28),
      'email' : ['shin12272014@gmail.com'],
      'email_on_failure' : True,
      'owner' : 'admin'
  }

  with DAG( dag_id = 'tutorial' , 
            default_args = default_args, 
            description = 'A simple tutorial DAG',
            schedule_interval = timedelta(days=1),
            start_date = datetime(2022, 1, 28 ),
            tags = ['example'] 
          ) as dag :
  ```


## Operator 종류
- Operator의 종류는 매우 다양하다. 어떤 것을 주로 사용하는지는 잘 모르겠으나 친절하게 아래의 페이지에 정리가 잘 되어있다. 주로 사용될 것으로 보이는 Operator 몇가지에 대해서만 정리해보자.
  - 링크 : https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/index.html


### BashOperator
  ```python
  t1 = BashOperator ( 
          task_id = 'sleep_date', 
          bash_command = 'echo 1' )
  ```
