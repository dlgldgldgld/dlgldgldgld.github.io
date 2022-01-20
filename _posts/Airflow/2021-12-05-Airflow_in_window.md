---
layout: single
title:  "Window에서 Airflow 설치해보기"
category: Airflow
tag: Airflow
---

# Airflow for Window Subsystem for Linux

Airflow Linux 환경을 기반으로 설치가 되어 있는지 Window 환경에서는 진행이 되지 않았다.

Window에서는 WSL이나 Docker를 사용해서 환경을 구성해야 하는데, 이 글에서는 WSL로 진행을 하는 방법에 대해서 소개한다.

## 설치 방법

1. Microsoft Store에서 Ubuntu를 검색 후, 설치. 그리고 재시작
2. **cmd를 열고 wsl 을 입력**하거나, **Ubuntu application을 실행**하면 Ubuntu 전용 Shell을 사용할 수 있다. 두 방법 중 하를 통해 Terminal을 실행시킨다.
3. 이 후, Ubuntu 업데이트를 진행한다.
    
    ```python
    sudo apt update && sudo apt upgrade 
    ```
    
4. pip3 를 업데이트 한다.
    
    ```
    sudo apt-get install software-properties-common
    sudo apt-add-repository universe
    sudo apt-get update && sudo apt-get install python3-pip
    ```
    
5. pip를 통해 apache-airflow 를 설치
    
    ```python
    pip3 install apache-airflow
    ```
    
6. `sudo nano /etc/wsl.conf` 를 입력한 후에 아래의 내용을 업데이트 한다.
    - ctrl + s , ctrl + x : 저장 후 종료
    
    ```python
    [automount] 
    root = / 
    options = "metadata"
    ```
    
7. 다음은 `nano ~/.bashrc`를 친 다음 AIRFLOW_HOME 환경 변수를 설정한다.
    - 원본 문서의 경우 `export AIRFLOW_HOME=/c/Users/philipp/AirflowHome` 로 되어있는데, 경로 인식이 되지 않는 오류가 발생한다.
        - Permission Error no 1 , Permission Error no 15 였던걸로 기억함.
            - 이는 `/home/username` 경로에 넣으면 해결됨.
            
            ![alt]({{ site.url }}{{ site.baseurl }}/assets/images/2021-12-05-Airflow_in_window/Untitled.png)
            
    
8. 설정이 끝났다면 Terminal을 닫고 다시 재접속한다.
9. 그다음 `airflow info` 명령어를 실행
    1. 정상 설치가 되었다면 아래 이미지와 같이 실행. 
        - package missing error 가 발생할 수도 있음.
            - 메시지가 있을테니 확인 후 설치 진행.
        
        ![alt]({{ site.url }}{{ site.baseurl }}/assets/images/2021-12-05-Airflow_in_window/Untitled%201.png)
        
10. `airflow db init` 명령어를 실행해서 초기화를 진행 
    1. 완료되면 .db 파일 하나랑 py 파일이 생성
        1. db는 sqlite3 로 되어있는데, 실행에 필요한 환경이 설정되있는 듯 하다.
            - dag 파일이랑 위치 등등..
    
     
    
11. `airflow webserver -p 8080` 를 실행하면 아래와 같이 주소가 뜨고, 해당 주소로 접속하면 airflow server가 생긴다.
    - 만약 0.0.0.0:8080 으로 접속이 되지 않는다면 ***localhost:8080*** 으로 접속.
    
    ![alt]({{ site.url }}{{ site.baseurl }}/assets/images/2021-12-05-Airflow_in_window/Untitled%202.png)
    
    ![alt]({{ site.url }}{{ site.baseurl }}/assets/images/2021-12-05-Airflow_in_window/Untitled%203.png)
    
12. 페이지에 접속을 하기 위해서는 계정 생성이 필요한데 아래와 같은 양식으로 계정을 하나 생성함.
    - `airflow users create --help` 명령어를 입력하면 생성 방법에 대한 설명이 있음.
        
        ![alt]({{ site.url }}{{ site.baseurl }}/assets/images/2021-12-05-Airflow_in_window/Untitled%204.png)
        

> 참고 문헌
> 
- Setting Airflow on WSL

[Run Apache Airflow on Windows 10 without Docker](https://towardsdatascience.com/run-apache-airflow-on-windows-10-without-docker-3c5754bb98b4)

- Airflow db Init 진행시 에러 대응 사항.

[Airflow PythonVirtualenvOperator, No such file or directory: 'virtualenv'](https://stackoverflow.com/questions/65295957/airflow-pythonvirtualenvoperator-no-such-file-or-directory-virtualenv)

- Airflow webserver 진행시 `[airflow-webserver.pid](http://airflow-webserver.pid)` 에 권한이 없다고 에러 발생시

[Airflow webserver error in WSL: PermissionError: [Errno 1] Operation not permitted:](https://stackoverflow.com/questions/69489623/airflow-webserver-error-in-wsl-permissionerror-errno-1-operation-not-permitt)

- Airflow 환경 구성 Youtube

[Apache Airflow 2.0 Tutorial for Beginners - Part 5: How to write our first airflow DAG!](https://www.youtube.com/watch?v=CLkzXrjrFKg)