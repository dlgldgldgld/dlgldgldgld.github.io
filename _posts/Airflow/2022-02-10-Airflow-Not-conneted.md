---
layout: single
title:  "WSL에서 Airflow 접속이 되지 않을때"
category: Airflow
tag: Airflow
---

개인 프로젝트를 위해 WSL에서 Airflow를 사용 중인데 잘 되던 것이 갑자기 되지 않았다. 현상은 localhost:port로 접속하면 아무런 반응이 없는 것이다.
cmd에서 `telnet localhost port` 을 해도 별다른 반응이 없다.

이를 해결하기 위해 구글링을 해본 결과 문제는 생각보다 간단하게 해결되었다.


[ 해결 방법 ]
1) `wsl --shutdown` 으로 wsl 종료
2) 다시 재접속 후, airflow 실행시 정상 수행.

출처 : https://github.com/microsoft/WSL/discussions/2471
