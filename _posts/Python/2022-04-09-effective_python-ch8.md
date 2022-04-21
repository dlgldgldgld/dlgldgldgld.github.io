---
layout: single
title:  "[파이썬 코딩의 기술 : Effective PYTHON 2ND 요약 및 코드 정리] CHAPTER 8. 강건성과 성능"
category: Python
tag: Python
---

## 65. try/except/else/finally의 각 블록을 잘 활용하라

### finally 블록
- try에 들어가서 except를 하더라도 무조건 실행되는 block.
- 주로 파일 핸들을 안전하게 닫기 위해 사용되는 문법이다.

```python
try:
    file = open("test.txt","w")
finally:
    file.close()
```

### else 블록
- try에서 예외를 발생시키지 않으면 실행되는 블록
- 아래 예제와 같이 코드를 분리하여 가독성을 높이고자 할때 사용되는 방식이다.

```python
import json

def load_json_key(data, key):
    try:
        print('* JSON 데이터 읽기')
        result_dict = json.loads(data)
    except ValueError as e:
        print('* ValudError 처리')
        raise KeyError(key) from e
    else:
        print('* 키 검색')
        return result_dict[key]
```

## 66. 재사용 가능한 try/finally 동작을 원한다면 contextlib과 with문을 활용하라

python에서는 with문과 같이 특정 문맥(context) 안에서만 작업을 수행하는 기능을 제공한다.

```python
with open("filename.txt", "w") as handler:
    handler.write("hello")
```

이 같은 기능은 class 내부에 `__enter__`, `__exit__` 함수를 구현해서 `with`문 안에 포함시키면 동작이 가능하다.  
이 방식의 단점은 with 문에서 에러가 발생할 경우 `__exit__` 함수의 구현이 까다로울 수 있다는 것인데 이를 보완하기 위해 나온 것이 contextlib 이다.

```python
import logging
from contextlib import contextmanager

@contextmanager
def debug_logging(level):
    logger = logging.getLogger()
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        yield
    finally:
        logger.setLevel(old_level)

def my_function():
    logging.debug('디버깅')
    logging.error('에러 로그')
    logging.debug('디버깅')

with debug_logging(logging.DEBUG):
    print("* 내부:")
    my_function()

print('* 외부:')
my_function()
```

위의 코드와 같이 간단하게 구현이 가능하다. 저렇게 source를 작성하면 with문에서 쉽게 동작하게 된다.  
with -> yield -> my_function -> finally 순으로 소스가 동작한다.

그래서 해당장에서 말하고자 하는 것은 **try/finally 구문을 여러곳에서 사용할 곳이 필요하다면 with문과 contextmanager를 사용해서 쉽게 구현을 할 것**을 권장하는 것이다.

## 67. 지역 시간에는 time보다는 datetime을 사용하라

프로그램에서 여러 나라의 Local time을 다뤄야 한다면 time 대신 datetime을 사용할 것을 권장한다.

예를 들어 time을 사용해 태평양 시간대와 한국 시간대를 동시에 다루고 싶다고 하자.  

```python
import os
if os.name == 'nt': # window
    print("It can't execute in window OS.")
else:
    parse_format = "%Y-%m-%d %H:%M:%S %Z'
    depart_icn = '2020-08-27 19:13:04 KST'
    time_tuple = time.strptime(depart_icn, parse_format)
    time_str = time.strftime(time_format, time_tuple)
    print(time_str)

    arrival_sfo = '2020-08-28 04:13:04 PDT'
    time_tuple = time.strtime(arrival_sfo, parse_format) 
    
>>> 2020-08-27 19:13:04
>>> Traceback ...
>>> Value Error: unconverted data remains: PDT
```

time 모듈을 통해 여러 시간대를 다루면 위와 같이 ValueError가 발생한다.  
이는 환경에 따라 시간대를 지원해주지 않을 수 있기 때문에 신뢰되는 작업이 아니다.

그래서 이를 통일하기 위해 datetime, pytz를 통해 여러 지역시간을 다룰 것을 권장하고 있다. 

datetime을 사용하면 아래 소스와 같이 간단하게 UTC를 KST로 바꾸는 것이 가능하다.

```python
from datetime import datetime, timezone

now = datetime(2020, 8, 27, 10, 13, 4)
now_utc = now.replace(tzinfo.timezone.utc)
now_local = now_utc.astimezone()
print(now_local)
```

timestamp로 바꾸는 것 또한 간편하다.

```python
time_str = '2020-08-27 19:13:04'
now = datetime.strptime(time_str, time_format)

time_tuple = now.timetuple()
utc_now = time.mktime(time_tuple)

print(utc_now)
```

pytz를 사용하면 간편하게 여러 시간대를 전환할 수 있다.

```python
import pytz

arrival_sfo = '2020-08-28 04:13:04'
sfo_dt_naive = datetime.strptime(arrival_sfo, time_format)   # 시간대가 설정되지 않은 시간
eastern = pytz.timezone('US/Pacific')                        # 샌프란시스코의 시간대
sfo_dt = eastern.localize(sfo_dt_naive)                      # 시간대를 샌프란시스코 시간대로 변경
utc_dt = pytz.utc.normalize(sfo_dt.astimezone(pytz.utc))     # UTC로 변경
print(utc_dt)

korea = pytz.timezone('Asia/Seoul')
korea_dt = korea.normalize(utc_dt.astimezone(korea))
print(korea_dt)

nepal = pytz.timezone('Asia/Katmandu')
nepal_dt = nepal.normalize(utc_dt.astimezone(nepal))
print(nepal_dt)
```

## 68. copyreg을 사용해 pickle을 더 신뢰성 있게 만들라.

pickle은 python 객체를 직렬화해서 bin 형태로 저장을하기 위해 제공되는 python module이다.  
이 모듈의 단점중 하나는 버전이 서로 다른 상태에서 데이터를 load한 경우 신규 버전의 변수가 반영이 되지 않는다는 점이 있다.

예를들면 A version 에서는 GameState라는 클래스의 변수가 level, lives 밖에 없었다면 B Version에서는 points가 추가되었다고 하자.  
B Version에서 A Version의 bin 파일을 load하면 points 변수는 포함되지 않은 상태로 객체가 역직렬화된다.

이러한 문제들로 인해 copyreg이라는 모듈을 사용해서 신뢰성을 보장하게 한다.


```python
import pickle


class GameState:
    def __init__(self):
        self.level = 0
        self.lives = 4
        ## self.points = 0


state = GameState()
state_path = "game_state.bin"
with open(state_path, "wb") as f:
    pickle.dump(state, f)

with open(state_path, "rb") as f:
    state_after = pickle.load(f)
    print(state_after.__dict__)


class GameState:
    def __init__(self):
        self.level = 0
        self.lives = 4
        self.points = 0  # 추가됨.


with open(state_path, "rb") as f:
    state_after = pickle.load(f)
    print("class에 변수를 추가하더라도 load시 반영되지 않음.")
    print(state_after.__dict__)

import copyreg


class GameState:
    def __init__(self, level=0, lives=4, points=0):
        self.level = level
        self.lives = lives
        self.points = points  # 추가됨.


def unpickle_game_state(kwargs):
    return GameState(**kwargs)


def pickle_game_state(game_state):
    kwargs = game_state.__dict__
    return unpickle_game_state, (kwargs,)


copyreg.pickle(GameState, pickle_game_state)
state = GameState()
state.points += 1000
serialized = pickle.dumps(state)
state_after = pickle.loads(serialized)
print(state_after.__dict__)


class GameState:
    def __init__(self, level=0, lives=4, points=0, magic=5):
        self.level = level
        self.lives = lives
        self.points = points  # 추가됨.
        self.magic = magic


print("이전:", state.__dict__)
state_after = pickle.loads(serialized)
print("이후:", state_after.__dict__)

```

```text
{'level': 0, 'lives': 4}
class에 변수를 추가하더라도 load시 반영되지 않음.
{'level': 0, 'lives': 4}
{'level': 0, 'lives': 4, 'points': 1000}
이전: {'level': 0, 'lives': 4, 'points': 1000}
이후: {'level': 0, 'lives': 4, 'points': 1000, 'magic': 5}
```

unpickle_game_state, pickle_game_state 을 변경하면 Class Version 지정도 가능하다.

```
def unpickle_game_state(kwargs):
    version = kwargs.pop('version', 1)
    if version == 1:
        del kwargs['lives']
    return GameState(**kwargs)


def pickle_game_state(game_state):
    kwargs = game_state.__dict__
    kwargs['version'] = 2
    return unpickle_game_state, (kwargs,)
```
class 이름이 변경된 경우에 unpickle 하면 변환을 하지 못하는 오류가 있다.
이것 또한 copyreg을 통해 해결이 가능하다. 

```python
class BetterGameState:
    def __init__(self, level=0, points=0, magic=5):
        self.level = level
        self.points = points
        self.magic = magic

pickle.loads(serialized)
# GameState를 찾을수 없으므로 Error 발생.

copyreg.pickle(BetterGameState, picle_game_state)
state = BetterGameState()
serialized = pickle.dumps(state)
print(serialized)
```

## 69. 정확도가 매우 중요한 경우에는 decimal을 사용하라  

IEEE 754 부동소수점 수의 내부(이진) 표현법으로 인해 일반 정수는 올바른 결과를 내보내지 않을 확률이 높다.  
이런 경우 Decimal을 사용하면 좀 더 정교하게 계산이 가능하다.

```python
rate = 1.45
seconds = 3*60 + 42
cost = rate * seconds / 60
print(cost)

from decimal import Decimal
rate = Decimal('1.45')
seconds = Decimal(3 * 60 + 42)
cost = rate * seconds / Decimal(60)
print(cost)


# 소수점을 넘길시에는 문자열을 사용하면 오차가 발생하지 않음.
print(Decimal('1.45'))
print(Decimal(1.45))

# 정수는 제외
print(Decimal('456'))
print(Decimal(456))

# 또다른 예제
rate = Decimal('0.05')
seconds = Decimal('5')
small_cost = rate * seconds / Decimal(60)
print(small_cost)

from decimal import ROUND_UP

rounded = cost.quantize(Decimal('0.01'), rounding=ROUND_UP)
print(f'반올림 전: {cost} 반올림 후: {rounded}')

rounded = small_cost.quantize(Decimal('0.01'), rounding=ROUND_UP)
print(f'반올림 전: {small_cost} 반올림 후: {rounded}')
```

```text
5.364999999999999
5.365
1.45
1.4499999999999999555910790149937383830547332763671875
456
456
0.004166666666666666666666666667
반올림 전: 5.365 반올림 후: 5.37
반올림 전: 0.004166666666666666666666666667 반올림 후: 0.01
```

## 70. 최적화하기 전에 프로파일링을 하라

프로파일링이란 코드의 성능을 측정하는 것이다.  
예를 들자면 특정 함수가 전체 프로그램 수행시간 중 몇 % 를 차지하고 있는지 통계를 내보는 등의 행위를 의미한다.

Python에서는 이에 관한 모듈이 제공된다. cProfile 모듈이 이에 해당된다.  
아래는 해당 모듈을 사용해서 프로파일링한 소스의 예제이다.  

Profile class를 생성한 후, runcall 함수를 호출해서 인자로 프로파일링을 해볼 함수명을 입력하면 된다.  
이 후, pstats 모듈의 Stats 클래스를 사용해서 결과를 출력하는 것이 가능하다.

만약 특정 함수가 여러 function에서 실행 될 경우, Caller 별로 통계치를 보고 싶을 수 있다.  
이는 **print_callers()** 함수를 통해 확인이 가능하다.

```python
def my_utility(a, b):
    c = 1
    for i in range(100):
        c += a * b

def first_func():
    for _ in range(1000):
        my_utility(4, 5)

def second_func():
    for _ in range(10):
        my_utility(1, 3)

def my_program():
    for _ in range(20):
        first_func()
        second_func()

from cProfile import Profile
from pstats import Stats

profiler = Profile()
profiler.runcall(my_program)

stats = Stats(profiler)
stats.strip_dirs()
stats.sort_stats('cumulative')
stats.print_stats()

stats.print_callers()
```

```text
         20242 function calls in 0.109 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.109    0.109 hello.py:14(my_program)
       20    0.003    0.000    0.108    0.005 hello.py:6(first_func)
    20200    0.105    0.000    0.105    0.000 hello.py:1(my_utility)
       20    0.000    0.000    0.001    0.000 hello.py:10(second_func)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}


   Ordered by: cumulative time

Function                                          was called by...
                                                      ncalls  tottime  cumtime
hello.py:14(my_program)                           <-
hello.py:6(first_func)                            <-      20    0.003    0.108  hello.py:14(my_program)
hello.py:1(my_utility)                            <-   20000    0.104    0.104  hello.py:6(first_func)
                                                         200    0.001    0.001  hello.py:10(second_func)
hello.py:10(second_func)                          <-      20    0.000    0.001  hello.py:14(my_program)
{method 'disable' of '_lsprof.Profiler' objects}  <-

```

## 71. producer-consumer 전용 queue로는 deque를 사용하라

프로그램 개발시 흔히 사용되는 패턴인 producer-consumer pattern을 구현할때 task를 관리하기 위해 queue를 사용한다.  
일부 개발자들은 queue를 list를 통해 구현하는데 이때 몇가지 문제가 발생한다.

1. 크기(cardinality)가 늘어날수록 리스트 타입의 성능은 선형보다 더 나빠진다.
2. list를 통해 pop(0)을 할때 리스트의 모든 원소를 재배열하기 때문에 느리다.

그로 인해 해당 챕터에서는 list를 사용하는 대신 deque를 사용할 것을 권장한다.  
append 시의 속도는 크게 차이가 없지만 pop을 할때는 무시할 수 없는 수치로 차이가 나게된다.  

책에서 측정된 성능을 정리하면 다음과 같다. 

| case          | list      | deque     | diff       |
| ------------- | --------- | --------- | ---------- |
| append(500)   | 0.000023s | 0.000022s | 0.000001s  |
| append(1,000) | 0.000045s | 0.000044s | 0.000001s  |
| append(2,000) | 0.000087s | 0.000091s | -0.000004s |
| append(3,000) | 0.000134s | 0.000142s | -0.000008s |
| append(4,000) | 0.000181s | 0.000192s | -0.000011s |
| append(5,000) | 0.000231s | 0.000244s | -0.000013s |
| pop(500)      | 0.000043s | 0.000019s | 0.000024s  |
| pop(1,000)    | 0.000097s | 0.000041s | 0.000056s  |
| pop(2,000)    | 0.000252s | 0.000081s | 0.000001s  |
| pop(3,000)    | 0.000464s | 0.000126s | 0.000171s  |
| pop(4,000)    | 0.000751s | 0.000169s | 0.000582s  |
| pop(5,000)    | 0.001229s | 0.000213s | 0.001016s  |

## 72. 정렬된 시퀀스를 검색할 때는 bisect를 사용하라

정렬된 시퀀스 컨테이너를 검색할때 보통 qsort를 사용해 binary search를 한다.  
python에서는 binary search를 위한 module이 이미 구현되어 있어 이를 사용해 시퀀스 검색을 빠르게 할 수 있다.

bisect_left, bisect_right 두 가지 존재한다.  

- bisect_left : sequence 안에 데이터가 존재시 값이 위치한 index return.
- bisect_right : sequence 안에 데이터가 존재시 index의 오른쪽을 return.   

```python
import random
import timeit
from bisect import bisect_left

size = 10**5
iterations = 1000

data = list(range(size))
to_lookup = [random.randint(0, size) for _ in range(iterations)]


def run_linear(data, to_lookup):
    for index in to_lookup:
        data.index(index)


def run_bisect(data, to_lookup):
    for index in to_lookup:
        bisect_left(data, index)


baseline = timeit.timeit(
    stmt="run_linear(data, to_lookup)", globals=globals(), number=10
)

print(f"선형 검색: {baseline:.6f}초")

comparison = timeit.timeit(
    stmt="run_bisect(data, to_lookup)", globals=globals(), number=10
)

print(f"이진 검색: {comparison:.6f}초")

slowdown = 1 + ((baseline - comparison) / comparison)
print(f"선형 검색이 {slowdown:.1f}배 더 걸림")

```

```text
선형 검색: 4.093573초
이진 검색: 0.006002초      
선형 검색이 682.1배 더 걸림
```

## 73. 우선순위 큐로 heapq를 사용하는 방법을 알아두라

priority queue는 특정 기준에 따라 queue의 순서를 항상 유지하고 싶을때 사용되는 
container 종류 중의 하나이다.  

책에서는 이를 위해 `heapq` 라는 모듈을 사용할 것을 권장한다.  
heapq 모듈은 list container 변수를 기반으로 실제 heap 처럼 push를 하거나 pop을 할 수 있게끔
확장성이 높은 기능등을 제공한다.

만약 자신이 직접 정의한 class를 element로 넣고 싶다면 `functools.total_ordering`을 데코레이터로 받은 다음  
magic method `__lt__`를 정의해주면 된다.

```python
from heapq import heappush, heappop, heapify

import functools


@functools.total_ordering
class Book:
    def __init__(self, title, due_date):
        self.title = title
        self.due_date = due_date

    def __lt__(self, other):
        return self.due_date < other.due_date


queue = []

heappush(queue, Book("harry porter", "2022-04-20"))
heappush(queue, Book("작은 아씨들", "2022-05-21"))

heappop(queue)  # 가장 앞의 책을 pop
heappop(queue)

queue = [
    Book("오만과 편견", "2020-06-01"),
    Book("타임 머신", "2020-06-02"),
    Book("죄와 벌", "2020-06-04"),
    Book("폭풍의 언덕", "2020-06-03"),
]

heapify(queue)

```

## 74. bytes를 복사하지 않고 다루려면 memoryview와 bytearray를 사용하라

python에서의 자료형은 대부분 자동으로 연역된다.  
이런 특성 때문인지 bytes 데이터를 다룰때는 기본적으로 복사가 발생하게 된다.  

만약 bytes 데이터를 slice 할 경우 대부분 copy가 발생하게 되는데 이런 copy를 방지하기 위해  
memoryview라는 모듈이 제공되고 있다.

이는 C-API의 Buffer protocol을 통해 구현이 되어 있으며 이를 사용해 zero copy 연산이 가능하도록 지원한다.
일반적인 사용 예시는 다음과 같다.  

책에서의 예시는 copy로 데이터를 읽어오는 방식보다 엄청나게 빠른 속도로 개선이 가능하다.  
( byte slice : 5밀리초, memoryview : 250 나노초 )

```python
data = "동해물과 abc 백두산이 마르고 닳도록".encode("utf8")
view = memoryview(data)
chunk = view[12:19]
print(chunk)
print(chunk.nbytes)
print(chunk.tobytes())
print(chunk.obj)
```

bytes를 memoryview에 보내면 값을 수정하지 못한다.  
실제로 bytes 자료형의 데이터를 index를 통해 값을 바꾸면 에러가 발생하게 된다.

그럴때는 bytearray를 사용하면 값을 바꾸는 것이 가능하다.  
이처럼 bytes 자료형을 사용할 경우 memoryview나 bytearray를 사용하는 것을 검토해보면 성능 향상을 도모할 수 있게 된다.

```python

# Python program to illustrate
# Modifying internal data using memory view
 
# random bytearray
byte_array = bytearray('XYZ', 'utf-8')
print('Before update:', byte_array)
 
mem_view = memoryview(byte_array)
 
# update 2nd index of mem_view to J
mem_view[2] = 74
print('After update:', byte_array)
```