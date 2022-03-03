---
layout: single
title:  "[파이썬 코딩의 기술 : Effective PYTHON 2ND 요약 및 코드 정리] CHAPTER 2. 리스트와 딕셔너리"
category: Python
tag: Python
---

# [파이썬 코딩의 기술 : Effective PYTHON 2ND 요약 및 코드 정리] CHAPTER 2. 리스트와 딕셔너리

## Chapter 2. 리스트와 딕셔너리 

### 11. 시퀀스를 슬라이싱하는 방법을 익혀라

**- Code**
```python
a =['a','b','c','d','e','f','g','h']
b = a[:20] # error 발생하지 않음.
# a[20] # error 

# unpacking을 할때는 slice의 개수가 맞아야 에러가 발생하지 않음.
c,d = a[:2]
print(f'{c=}, {d=}')

# 그러나 대입할때는 굳이 맞추지 않아도 된다.
print(f'이전 {a=}')
a[2:7] = [99,22,14]
print(f'이후 {a=}')

# 반대의 경우에도 가능하다.
print(f'이전 {a=}')
a[2:3] = [47,11]
print(f'이후 {a=}')


# 리스트의 복사본을 얻으려면 슬라이스를 사용해야 한다.
b = a[:]
b[0] = 56
print (f'{a=}, {b=}')

# 사용하지 않으면 주소값을 가져와서 값을 공유한다.
b = a
b[0] = 56
print (f'{a=}, {b=}')
```

**- Result**
```text
c='a', d='b'
이전 a=['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
이후 a=['a', 'b', 99, 22, 14, 'h']
이전 a=['a', 'b', 99, 22, 14, 'h']
이후 a=['a', 'b', 47, 11, 22, 14, 'h']
a=['a', 'b', 47, 11, 22, 14, 'h'], b=[56, 'b', 47, 11, 22, 14, 'h']
a=[56, 'b', 47, 11, 22, 14, 'h'], b=[56, 'b', 47, 11, 22, 14, 'h']
```

<br>

### 12. 스트라이드와 슬라이스를 한 식에 함께 사용하지 말라
스트라이드? 
- 리스트[시작:끝:증가값] 으로 일정한 간격을 슬라이싱 할 수 있는 구문

**- Code**
```python
# stride 문법 예시
a =['a','b','c','d','e','f','g','h']
odds = a[::2]
evens = a[1::2]
print(f'{a=}, {odds=}, {evens=}')

# 역순을 뽑고 싶을때, unicode나 byte의 경우에는 잘 나온다.
x = b'mongoose'
y = x[::-1]
print(f'{y=}')

x = u'한글'
y = x[::-1]
print(f'{y=}')

# 그러나 아래와 같이 encode, decode를 하는 경우에는 에러가 발생한다.
# 근데 이건 당연한거 아니냐?
w = u'세종대왕'
x = w.encode('utf-8')
y = x[::-1]
# decode 하는 과정에서 에러가 발생
#z = y.decode('utf-8')

# 음수가 -1 이 아닌 경우는 어떻게 동작할까?
x = ['a','b','c','d','e','f','g','h']
print ( f'{x=}')
print ( f'{x[::2]=}' )
print ( f'{x[::-2]=}' )

# 다음의 케이스들에 대해서 코드만 보고 결과를 바로 도출할 수 있을까..? 쉽지 않다.
# 그렇기 때문에 시작값이나 끝값이 포함된 상태해서 증가값을 포함시키 않는 것을 추천한다.
print(f'{x[2::2]=}')
print(f'{x[-2::-2]=}')
print(f'{x[-2:2:-2]=}')
print(f'{x[2:2:-2]=}')

```

**- Result**
```text
a=['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'], odds=['a', 'c', 'e', 'g'], evens=['b', 'd', 'f', 'h']
y=b'esoognom'
y='글한'
x=['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
x[::2]=['a', 'c', 'e', 'g']
x[::-2]=['h', 'f', 'd', 'b']
x[2::2]=['c', 'e', 'g']
x[-2::-2]=['g', 'e', 'c', 'a']
x[-2:2:-2]=['g', 'e']
x[2:2:-2]=[]
```

<br>

### 13. 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 이용하라

**- Code**
```python
# unpacking을 사용한 나머지 값들 load 방법
car_ages = [0,9,4,8,20,19,1,6,15]
car_ages_descending = sorted(car_ages, reverse=True)
oldest, second_oldest , other = car_ages_descending[0], car_ages_descending[1], car_ages_descending[2:]
print(f'{oldest=}, {second_oldest=}, {other=}')
oldest, second_oldest , *other = car_ages_descending
print(f'{oldest=}, {second_oldest=}, {other=}')

# 중간, 앞으로도 사용이 가능함
oldest, * other, youngest = car_ages_descending
print(f'{oldest=}, {other=}, {youngest=}')
*other, sec_youngest, youngest = car_ages_descending
print(f'{other=}, {sec_youngest=}, {youngest=}')

# 길이가 맞지 않아도 가능.
short = [1,2]
a,b,*other = short
print(f'{a=}, {b=}, {other=}')

# 제너레이터의 경우에도 사용이 가능하다.
# 아래 함수는 csv 형식의 record와 header를 가진 제너레이터다.
def generate_csv():
    yield('날짜', '제조사', '모델', '연식', '가격')
    yield('2022/01/01', 'LG', 'SM5', '5년', '5100원')
    yield('2022/01/02', 'LG', 'SM5', '5년', '5100원')
    yield('2022/01/03', 'LG', 'SM5', '5년', '5100원')
    yield('2022/01/04', 'LG', 'SM5', '5년', '5100원')
    yield('2022/01/05', 'LG', 'SM5', '5년', '5100원')

# 리스트로 변환 후 슬라이싱해서 처리하는 것이 가능하다만
all_csv_rows = list(generate_csv())
header = all_csv_rows[0]
rows = all_csv_rows[1:]
print(f'{header=}, {len(rows)=}')

# 언패킹을 통해 iterable 처리하는 것도 가능하다
it = generate_csv()
header, *rows = it
print(f'{header=}, {len(rows)=}')


```

**- Result**
```text
oldest=20, second_oldest=19, other=[15, 9, 8, 6, 4, 1, 0]
oldest=20, second_oldest=19, other=[15, 9, 8, 6, 4, 1, 0]
oldest=20, other=[19, 15, 9, 8, 6, 4, 1], youngest=0
other=[20, 19, 15, 9, 8, 6, 4], sec_youngest=1, youngest=0
a=1, b=2, other=[]
6
header=('날짜', '제조사', '모델', '연식', '가격'), len(rows)=5
header=('날짜', '제조사', '모델', '연식', '가격'), len(rows)=5
```

<br>

### 14. 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라
sort 를 사용할때 key로 lambda 함수를 정의해서 자유롭게 기준을 정할 수 있는 것에 대한 설명이다.
**- Code**
```python
# 예제 Class
class Tool :
    def __init__(self, name, weight):
        self.name = name
        self.weight = weight

    def __repr__(self):
        return f'Tool({self.name=}, {self.weight=})'

tools = [
    Tool('수준계', 3.5),
    Tool('해머', 1.25),
    Tool('드라이버', 0.5),
    Tool('끌', 0.25)
]

# migic function을 정의하지 않은 상태로 sort를 호출하면 에러가 발생한다.
# tools.sort()

# 이럴때는 key로 lambda 값을 정의해서 attribute에 직접 접근을 한다면 굳이 magic method를 사용하지 않아도 된다.
tools.sort(key = lambda x : x.name)
print(f'{tools=}')
tools.sort(key = lambda x : x.weight)
print(f'{tools=}')

# labmda에는 str.lower() 같은 것도 사용할 수도 있다.
places = ['home', 'work', 'New York', 'Paris']
places.sort()
print(f'대소문자 구분 : {places}')
places.sort(key = lambda x : x.lower())
print(f'대소문자 구분 x : {places}')

# 하나 이상의 attribute를 기준으로 넣고 싶다면 tuple을 사용한다.
power_tools=[
    Tool('드릴',4),
    Tool('톱',5),
    Tool('착암기',40),
    Tool('연마기',4),
]

power_tools.sort(key=lambda x : (x.weight, x.name))
print(f'{power_tools=}')

# 특정 attribute의 방향을 뒤집고 싶다면 음수를 사용하자
power_tools.sort(key=lambda x : (-x.weight, x.name))
print(f'{power_tools=}')

# 물론 모두 가능한 것은 아니니깐 참고. 아래 코드는 아래 발생.
# power_tools.sort(key=lambda x : (x.weight, -x.name), reverse = True)
# 이같은 경우는 어쩔 수 없이 2번 sort를 하는 수 밖에 없다.
power_tools.sort(key=lambda x : x.name)
power_tools.sort(key=lambda x : x.weight, reverse=True)
print(f'{power_tools=}')
```

**- Result**
```text
tools=[Tool(self.name='끌', self.weight=0.25), Tool(self.name='드라이버', self.weight=0.5), Tool(self.name='수준계
', self.weight=3.5), Tool(self.name='해머', self.weight=1.25)]
tools=[Tool(self.name='끌', self.weight=0.25), Tool(self.name='드라이버', self.weight=0.5), Tool(self.name='해머', self.weight=1.25), Tool(self.name='수준계', self.weight=3.5)]
대소문자 구분 : ['New York', 'Paris', 'home', 'work']
대소문자 구분 x : ['home', 'New York', 'Paris', 'work']
power_tools=[Tool(self.name='드릴', self.weight=4), Tool(self.name='연마기', self.weight=4), Tool(self.name='톱', 
self.weight=5), Tool(self.name='착암기', self.weight=40)]
power_tools=[Tool(self.name='착암기', self.weight=40), Tool(self.name='톱', self.weight=5), Tool(self.name='드릴', self.weight=4), Tool(self.name='연마기', self.weight=4)]
power_tools=[Tool(self.name='착암기', self.weight=40), Tool(self.name='톱', self.weight=5), Tool(self.name='드릴', self.weight=4), Tool(self.name='연마기', self.weight=4)]
```

<br>

### 15. 딕셔너리 삽입 순서에 의존할 때는 조심하라

**- Code**
```python

# 순서가 보장이 되지만 덕타이핑 특성을 이용해 문제가 발생하게 만들 수도 있다.
votes = {
    'otter' : 1281,
    'polar bear' : 587,
    'fox' : 863
}

def populate_ranks(votes, ranks):
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i

def get_winner(ranks):
    return next(iter(ranks))

# 의도한 동작.
ranks = {}
populate_ranks(votes, ranks)
print(ranks)
winner = get_winner(ranks)
print(winner)


# 덕타이핑을 이용해 속여보자.
from collections.abc import MutableMapping

class SortedDict(MutableMapping):
    def __init__(self):
        self.data = {}
    def __repr__(self):
        return f'{self.data}'
    def __getitem__(self, key):
        return self.data[key]
    def __setitem__(self, key, value) -> None:
        self.data[key] = value
    def __delitem__(self, key):
        del self.data[key]
    # iter을 호출할때 key 값을 기준으로 정렬을 시킨후 yield를 하면, 일반 dict과는 의도치 않은 동작으로 바꿔버릴 수 있다!
    def __iter__(self):
        keys = list(self.data.keys())
        keys.sort()
        for key in keys:
            yield key
    def __len__(self):
        return len(self.data)

sorted_rank = SortedDict()
populate_ranks(votes, sorted_rank)
print(sorted_rank)
winner = get_winner(sorted_rank)
print(winner)

# 보수하는 방법 소개
# 1. ranks 딕셔너리를 특정 순서로 iteration 되지 않다고 가정하고 구현해버리기

def get_winner(ranks):
    for name,rank in ranks.items():
        if rank == 1 :
            return name

# 2. dict 타입일때만 동작 시키게 만들기
def get_winner(ranks):
    if not isinstance(ranks ,dict) :
        return TypeError('dict 인스턴스가 필요합니다.')
    return next(iter(ranks))

# 3. typing 모듈을 사용해서 아예 정적으로 막아버리게 해라
# pip install mypy
# python -m mypy --strict hello.py 로 정적 검사시 에러 발생.
from typing import Dict, MutableMapping

def populate_ranks(votes: Dict[str,int],
                   ranks: Dict[str,int]) -> None:
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i

def get_winner(ranks: Dict[str,int])->str:
    return next(iter(ranks))

class SortedDict(MutableMapping[str,int]):
    def __init__(self):
        self.data = {}
    def __repr__(self):
        return f'{self.data}'
    def __getitem__(self, key):
        return self.data[key]
    def __setitem__(self, key, value) -> None:
        self.data[key] = value
    def __delitem__(self, key):
        del self.data[key]
    # iter을 호출할때 key 값을 기준으로 정렬을 시킨후 yield를 하면, 일반 dict과는 의도치 않은 동작으로 바꿔버릴 수 있다!
    def __iter__(self):
        keys = list(self.data.keys())
        keys.sort()
        for key in keys:
            yield key
    def __len__(self):
        return len(self.data)

sorted_rank = SortedDict()
populate_ranks(votes, sorted_rank)
print(sorted_rank)
winner = get_winner(sorted_rank)
print(winner)
```

**- Result**
```text
{'otter': 1, 'fox': 2, 'polar bear': 3}
otter
{'otter': 1, 'fox': 2, 'polar bear': 3}
fox
{'otter': 1, 'fox': 2, 'polar bear': 3}
fox
```

mypy로 정적 에러 검출 진행
```text
D:\test>python -m mypy --strict hello.py
hello.py:9: error: Function is missing a type annotation
...
hello.py:75: error: Argument "key" to "sort" of "list" has incompatible type overloaded function; expected "Callable[[str], Union[SupportsDunderLT, SupportsDunderGT]]"
hello.py:79: error: Name "get_winner" already defined on line 15
hello.py:82: error: Name "SortedDict" already defined on line 29
hello.py:83: error: Function is missing a return type annotation
hello.py:83: note: Use "-> None" if function does not return a value
hello.py:85: error: Function is missing a type annotation
hello.py:87: error: Function is missing a type annotation
hello.py:89: error: Function is missing a type annotation for one or more arguments
hello.py:91: error: Function is missing a type annotation
hello.py:94: error: Function is missing a type annotation
hello.py:99: error: Function is missing a type annotation
hello.py:102: error: Call to untyped function "SortedDict" in typed context
hello.py:103: error: Call to untyped function "populate_ranks" in typed context
hello.py:105: error: Call to untyped function "get_winner" in typed context
Found 34 errors in 1 file (checked 1 source file)
```

<br>

### in을 사용하고 딕셔너리 키가 없을 때 KeyError를 처리하기보다는 get을 사용하라

**- Code**
```python
votes = {
    '바게트' : ['철수', '순이'],
    '치아바타' : ['하니', '유리'],
}

# 일반 if 문의 경우 key를 2번 탐색함
key = '브리오슈'
who = '단이'

if key in votes: # key 첫번째 탐색
    names = votes[key] # key 두번째 탐색
else:
    votes[key] = names = [] # value 대입

# try를 쓰면 한번만 읽는 것이 가능함
try :
    names = votes[key] # key 첫번째 탐색
except KeyError :
    votes[key] = names = [] # 없을때만 value 대입

# get 함수를 사용하면 위보단 가독성이 좋게 코드를 짤 수 있음.
if ( names := votes.get(key) ) is None :
    votes[key] = names = []

# 이보다 더 간단한 것은 setdefault을 사용하는 것이나 함수명이 부적절해서 사용하는 것을 추천치 않음.
names = votes.setdefault(key,[])

# 그리고 이것은 잘못썻을때 의도치 않은 동작이 나올 위험이 있음
data = {}
key = 'foo'
value = []
data.setdefault(key,value)
print(f'이전:{data=}')
value.append('hello') # 값을 바꾸면 data에 영향이 가기 때문에 매번 새로운 리스트를 생성해줘야 함.
print(f'이후:{data=}')

# 그리고 또 다른 단점으로는 아래의 케이스에서 불필요한 대입을 한번 더 하는 경우가 발생한다.
counters={
    "품퍼니켈" : 2, 
    "사워도우" : 1,
}

# list가 아니라서 깊은 복사가 진행.
value = counters.setdefault(key, 0) # value = counter[key] = 0
value = counters.get(key, 0) # value = 0, counters.get은 key가 없으면 그냥 0을 반환함.
counters = value + 1

print(f'{counters=}')
```

**- Result**
```text
이전:data={'foo': []}
이후:data={'foo': ['hello']}
counters=1
```

<br>

### 17. 내부 상태에서 원소가 없는 경우를 처리할 때는 setdefault보다 defaultdict를 사용하라

**- Code**
```python

class Visits:
    def __init__ (self):
        self.data = {}
    def __repr__(self):
        return f'{self.data}'
    def add(self, country, city):
        self.data.setdefault(country,set()).add(city)
        # 있든 없든 setdefault 에서 매번 set()을 호출하기 때문에 효율적이지 못함.

visits = Visits()
visits.add('러시아', '예카테린부르크')
visits.add('탄자니아', '잔지마르')
print(visits)

from collections import defaultdict
class Visits:
    def __init__ (self):
        self.data = defaultdict(set)
    def __repr__(self):
        return f'{self.data}'
    def add(self, country, city):
        self.data[country].add(city)
        # 이건 위처럼 그런 케이스가 아닌가봄, 그래서 defaultdict을 써야된다함.

visits = Visits()
visits.add('러시아', '예카테린부르크')
visits.add('탄자니아', '잔지마르')
print(visits)
```

**- Result**
```text
{'러시아': {'예카테린부르크'}, '탄자니아': {'잔지마르'}}
defaultdict(<class 'set'>, {'러시아': {'예카테린부르크'}, '탄자니아': {'잔지마르'}})
```

<br>

### 18. __missing__을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라

**- Code**
```python
# 이 방법도 좋지만, 딕셔너리를 많이 읽고 코드가 길어지는 단점이 있다.
pictures={}
path = 'profile_1234.png'

if (handle := pictures.get(path)) is None:
    try:
        handle = open(path, 'a+b')
    except OSError:
        print(f'경로를 열 수 없습니다: {path}')
        raise
else:
    pictures[path]=handle

handle.seek(0)
image_data = handle.read()

# 아래의 코드는 문제가 많다. 
# 우선 매번 setdefault를 할때마다 open을 호출하기 때문에, 
# 만약 이미 열려있던 핸들이 있다면 이를 없애버린다. 그래서 사용할 수 없다.
try:
    handle = pictures.setdefault(path, open(path, 'a+b'))
except OSError:
    print(f'경로를 열 수 없습니다: {path}')
    raise
else:
    handle.seek(0)
    image_data = handle.read()

# defaultdict을 사용해서 구현을 하려고 하지만 open_picture은 인자를 전달받을 수 없다.
# 그래서 defaultdict은 호출할 방법이 없다.
from collections import defaultdict

def open_picture(profile_path):
    try:
        return open(profile_path, 'a+b')
    except OSError:
        print(f'경로를 열 수 없습니다: {path}')
        raise

# pictures = defaultdict(open_picture)
# handle = pictures[path]
# handle.seek(0)
# image_data = handle.read()

# 파이썬에서는 이런 상황이 많기 때문에 __missing__ 이라는 특별 메소드를 제공한다.
# 만약 키가 없는 경우 이 함수로 들어가서 대응을 진행한다. 따라서 아래와 같이 클래스를 작성하면 깔끔하게 처리할 수 있다.

class Pictures(dict):
    def __missing__(self,key):
        value = open_picture(key)
        self[key] = value
        return value

pictures = Pictures()
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
```

**- Result**
```text

```

<br>


