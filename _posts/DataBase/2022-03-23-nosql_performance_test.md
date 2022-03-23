---
layout: single
title:  "[DataBase][NoSQL] 분산 KVS, 와이드 컬럼 스토어, 도큐먼트 스토어, 검색엔진에 대해서 알아보자 - 05. Performance 테스트"
category: [Database, NoSQL]
tag: [Database, NoSQL]
---

앞선 글까지 NoSQL의 4가지 종류(분산 KVS, WCS, Document Store, Search Engine)에  
대해서 살펴보았다. 각 문서에서는 RDBMS와 비교했을시 NoSQL의 성능이 더 우수하다고  
되어있지만 글로만 봐서는 체감이 되질 않는다.

그래서 이제는 실제로 얼만큼의 성능의 차이가 얼마만큼 나는지 체크를 해보고자 한다.

이번 POST에서는 **RDB(SQLITE) + NoSQL(분산 KVS, WCS, Document Store, Search Engine)간에 성능 테스트**를 주제로 글을 작성해 보고자 한다. 

다양한 케이스별로 얼마만큼의 성능이 나오는지 간단하게 체크를 해보자.

# 1. RDBMS vs Key-Value Store 성능 비교

첫번째로 Key-Value Store와 RDBMS 사이에 어떠한 차이가 있는지에 대해서 비교를 해보고자 한다.  
Key-Value Store의 장점은 <u>Key별로 빠르게 데이터를 Read 하거나 Write를 할 수 있다는 점</u>이다.  
예를 들면 유저정보와 같이 Key를 통해서 데이터를 빠르게 주고받아야 할때 강점을 발휘한다.

과연 얼마나 차이가 날까? 한번 Test 해보자.  
RDBMS는 SQLITE, Key-Value Store는 Apache Cassandra를 사용해 Test를 진행한다.  
( 앞서 소개한 KVS인 DynamoDB는 요금 걱정때문에 우선은 제외.. )  

유저 정보에 관한 Schema는 아래와 같이 정의해보았다.  

## 유저 정보(client_info) Schema

|Column|Column(eng)|Description|Type|범위|
|---|---|---|---|---|
|고객 id|user_id|고객의 id|text|uuid_4|
|고객 pw|user_pw|고객의 pw|text|12자리 알파벳 + 숫자의 조합|
|고객 성명|user_name|고객의 이름|text|-|
|회원/비회원|is_member|고객이 회원인지 비회원인지?|boolean|0 = 비회원, 1 = 회원|
|성별|sex|고객의 성별|boolean|0 = 여성, 1 = 남성|
|나이|age|고객의 나이|int|12~70|
|직업|job|고객의 직업에 대한 정보|text|-|
|주소|home_address|배송이 이뤄져야할 위치에 대한 정보|text|-|

## Test 1. 유저 정보 Read / Write

실시간 웹 서비스에서 유저의 정보를 빠르게 받아오고 가져오는 상황이 일어났다고 하자.  
첫번째로 알아보고 싶은 것은 실제 Write 속도가 얼마나 차이가 있을지에 대한 차이다.  

아래의 테스트 환경으로 진행해보자.

### Environments
- PC 사양
  - CPU - AMD Ryzen 5 3500X 6-Core Processor(3.60 GHz)
  - RAM - 16GB
  - OS - Window 10 Home
- DB Engine : SQLITE3( Local ), Cassandra( Local )
- **1000만명의 유저**가 있다고 가정하고 Read/Write Test 진행.
- Key = 고객 id(user_id)


### Test 1-1. Write Test ( 1000만 record )
SQLITE에는 Index를 걸었을때와 걸지 않았을때를 모두 테스트 해보았다.  
결과는 예상대로 SQLITE3(Index)에 Write를 했을때가 가장 느렸다.  

SQLITE3에 Index를 걸지 않았을때와 걸었을때는 약 5배정도로 Index를 걸었을때 Write하는게 느렸고,  
Cassandra와 SQLITE3(Index)의 차이는 SQLITE3(Index)가 **약 1.62배** 느리게 측정되었다. 

만약 실제 운영환경이라면 SQLITE3(Index)가 쓰이게 될 것이므로 확실히 이 결론만 놓고 본다면  
KVS가 Write 하는 것에는 더 좋다고 볼 수 있을 것 같다. 

물론 쿼리 최적화를 했을때는 또 다르게 결과가 나올 수 있겠지만 일단은 KVS가 더 빠른걸로 결론을 내자.

#### Test 1-1. Write Test Result

|Case|소요시간(sec)|image|
|----|----|----|
|SQLITE3 Write(No Index)|73.9409 seconds|![alt](../../../assets/images/2022-03-18-nosql_performance_test/TEST1_SQLITE3_WRITE_NOINDEX.png)|
|SQLITE3 Write(Index)|354.5286 seconds|![alt](../../../assets/images/2022-03-18-nosql_performance_test/TEST1_SQLITE3_WRITE_INDEX.png)|
|CASSANDRA Write|218.628 seconds|![alt](../../../assets/images/2022-03-18-nosql_performance_test/TEST1_CASSANDRA_MULTI_PROCESS_5.png)|

----

### Test 1-2. Read Test ( 1 record )
다음은 Read Test를 진행해보았다. read test시에는 속도가 거의 비슷하게 나왔다.  
sqlite3에서는 1~4 ms를 유지하고, Cassandra에서는 3ms를 유지하고 있다.

Read 속도는 큰 차이가 없는 것으로 확인된다.

`Query : select * from client_info where user_id = '7888a4bc-51c8-42a3-9f73-2b898a0a14e6'`


#### Test 1-2. Read Test Result

|Case|소요시간(sec)|image|
|----|----|----|
|SQLITE3 Read 1 record(No Index)|1760 ms|![alt](../../../assets/images/2022-03-18-nosql_performance_test/TEST1_2_SQLITE_NOINDEX.png)|
|SQLITE3 Read 1 record(Index)|1~4 ms|![alt](../../../assets/images/2022-03-18-nosql_performance_test/TEST1_2_SQLITE_INDEX.png)|
|CASSANDRA Read 1 record|3 ms|![alt](../../../assets/images/2022-03-18-nosql_performance_test/TEST1_2_CASSANDRA.png)|