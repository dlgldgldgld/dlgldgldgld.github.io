---
layout: single
title:  "[Airflow] - Airflow에서 DAG을 인식하는 방식"
category: Airflow
tag: Airflow
---

Airflow를 처음 사용할 때 DAG파일을 작성해도 인식이 안되는 문제를 종종 접하곤 했다.

문제 해결을 위해 공식 doc를 찾아보았고, [https://airflow.apache.org/docs/apache-airflow/stable/concepts/dags.html#loading-dags](https://airflow.apache.org/docs/apache-airflow/stable/concepts/dags.html#loading-dags)에 의하면 DAG은 파일 내에서 Global 변수로 선언되어야 Load를 한다고 명시가 되어있다. 

그냥 이 정도로만 알고있는 찰나 오픈채팅방에서 해당 주제가 다시 나오게 되었는데, 최근 파이썬을 깊게 사용해보고자 공부를 하고 있는 도중이라 Airflow에서 이를 어떻게 Python으로 구현하였는지 궁금해하지 않을 수가 없었다. 

그래서 오늘의 주제는 Airflow - DAG 인식하는 방법에 대해 코드리뷰를 진행해보고자 한다. 

# 1. DAG 처리하는 부분 소스에서 찾아보기!

일단 Airflow 자체를 디버깅하는 방법은 몰라서 DAG을 발견하지 못하면 나오는 로그를 통해 그 부분부터 찾아보는 것으로 시작을 했다.

Warning 로그는 다음과 같으며, `WARNING - No viable dags retrieved from "filepath"` 이 부분을 소스에서 한번 찾아보자.

해당 부분은 `airflow\dag_processing\processor.py` 파일안에 숨어있었다!
아래 소스를 보면 dagbag 이라는 인스턴스에 dags의 len을 구하고 값이 0 이하이면 WARNING을 찍게 구현하고 있음을 알 수 있다.

dagbag은 바로 윗부분의 try 구문쪽에서 생성하고 self._deactivate_missing_dags에도 전달을 한다. 
저 부분만 봤을때는 두곳중 하나에 dags를 채워주고 있을 것이 틀림없으므로 차례로 살펴보도록 하자.

**airflow\dag_processing\processor.py - LINE 624**
```python
    try:
        dagbag = DagBag(file_path, include_examples=False, include_smart_sensor=False)
    except Exception:
        self.log.exception("Failed at reloading the DAG file %s", file_path)
        Stats.incr('dag_file_refresh_error', 1, 1)
        return 0, 0

    self._deactivate_missing_dags(session, dagbag, file_path)
        
    if len(dagbag.dags) > 0:
        self.log.info("DAG(s) %s retrieved from %s", dagbag.dags.keys(), file_path)
    else:
        self.log.warning("No viable dags retrieved from %s", file_path)
        self.update_import_errors(session, dagbag)
        return 0, len(dagbag.import_errors)
```

dagbag의 생성자를 먼저 살펴보도록 하자. dags 변수는 line 27에서 최초로 dict 으로 선언이 된다. dags을 모으는 부분이 어딘가에 있을텐데.. 소스를 보니 LINE 40 번째에 `self.collect_dags` 라는 함수가 실행이 되고 있다. 누가봐도 저기서 dags을 모으는 것 같으니 일단 저기부터 확인해보도록 하자.

**airflow\models\dagbag.py - LINE 92**
```python
def __init__(
        self,
        dag_folder: Union[str, "pathlib.Path", None] = None,
        include_examples: bool = conf.getboolean('core', 'LOAD_EXAMPLES'),
        include_smart_sensor: bool = conf.getboolean('smart_sensor', 'USE_SMART_SENSOR'),
        safe_mode: bool = conf.getboolean('core', 'DAG_DISCOVERY_SAFE_MODE'),
        read_dags_from_db: bool = False,
        store_serialized_dags: Optional[bool] = None,
        load_op_links: bool = True,
    ):
        # Avoid circular import
        from airflow.models.dag import DAG

        super().__init__()

        if store_serialized_dags:
            warnings.warn(
                "The store_serialized_dags parameter has been deprecated. "
                "You should pass the read_dags_from_db parameter.",
                DeprecationWarning,
                stacklevel=2,
            )
            read_dags_from_db = store_serialized_dags

        dag_folder = dag_folder or settings.DAGS_FOLDER
        self.dag_folder = dag_folder
        self.dags: Dict[str, DAG] = {}
        # the file's last modified timestamp when we last read it
        self.file_last_changed: Dict[str, datetime] = {}
        self.import_errors: Dict[str, str] = {}
        self.has_logged = False
        self.read_dags_from_db = read_dags_from_db
        # Only used by read_dags_from_db=True
        self.dags_last_fetched: Dict[str, datetime] = {}
        # Only used by SchedulerJob to compare the dag_hash to identify change in DAGs
        self.dags_hash: Dict[str, str] = {}

        self.dagbag_import_error_tracebacks = conf.getboolean('core', 'dagbag_import_error_tracebacks')
        self.dagbag_import_error_traceback_depth = conf.getint('core', 'dagbag_import_error_traceback_depth')
        self.collect_dags(
            dag_folder=dag_folder,
            include_examples=include_examples,
            include_smart_sensor=include_smart_sensor,
            safe_mode=safe_mode,
        )
        # Should the extra operator link be loaded via plugins?
        # This flag is set to False in Scheduler so that Extra Operator links are not loaded
        self.load_op_links = load_op_links
```