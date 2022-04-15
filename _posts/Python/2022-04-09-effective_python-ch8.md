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