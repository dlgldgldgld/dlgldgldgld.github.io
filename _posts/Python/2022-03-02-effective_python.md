---
layout: single
title:  "파이썬 코딩의 기술 : Effective PYTHON 2ND - 요약 및 코드 정리"
category: Python
tag: Python
---

# 파이썬 코딩의 기술 : Effective PYTHON 2ND - 요약 및 코드 정리

## Chapter 1. 파이썬답게 생각하기 

### 01. 사용 중인 파이썬의 버전을 알아두라

**- Code**
```python
import sys
print(sys.version_info)
print(sys.version)
```

**- Result**
```text
sys.version_info(major=3, minor=9, micro=7, releaselevel='final', serial=0)
3.9.7 (default, Sep 16 2021, 16:59:28) [MSC v.1916 64 bit (AMD64)]
```

<br>

### 02. PEP-8 참고
pass 

<br>

### 03. bytes와 str의 차이를 알아두라 

**- Code**
```python
# b를 utf-8로 디코딩 하려면 decode 함수를 이용한다.
a = b'h\x65llo'
print(f'{list(a)=}')
print(f'{a=}')
a.decode('utf-8')

# python에서 str은 기본적으로 utf-8 형식이다. byte로 바꾸려면 encode 함수를 사용한다.
a = 'a\u0300 propos'
print(f'{list(a)=}')
print(f'{a=}')
# encode 함수의 기본 값은 utf-8이다.
default_encoding = a.encode()
print(f'{default_encoding=}')

utf8_encoding = a.encode('utf-8')
print(f'{utf8_encoding=}')

# 디폴트 인코딩은 locale 모듈에서 확인할 수 있다. 
# 현재 PC : cp949, wsl : UTF-8
# 디폴트 인코딩은 위의 encode 함수는 상관없고 file read / write 할때 적용이 된다.
import locale
print(locale.getdefaultlocale(), locale.getpreferredencoding())

# 아래와 같이 'w' 옵션으로 byte를 write하면 에러가 발생한다.
with open('data.bin','w') as f:
    # \x = hex code
    #f.write(b'\xf1\xf2\xf3\xf4\xf5')
    pass

# byte 데이터는 'wb' 옵션으로 사용하자. 'rb', 'r' 또한 마찬가지다.
with open('data.bin','wb') as f:
    f.write(b'\xf1\xf2\xf3\xf4\xf5')


```

**- Result**
```text
list(a)=[104, 101, 108, 108, 111]
a=b'hello'
list(a)=['a', '̀', ' ', 'p', 'r', 'o', 'p', 'o', 's']
a='à propos'
default_encoding=b'a\xcc\x80 propos'
utf8_encoding=b'a\xcc\x80 propos'
('ko_KR', 'cp949') cp949
```

<br>

### 04. f-문자열
pass

<br>

### 05. 복잡한 식을 쓰는 대신 도우미 함수를 작성하라
**- Code**
```python
from urllib.parse import parse_qs

# query string인 경우, parse_qs 라이브러리를 사용하자
my_values= parse_qs('빨강=5&파랑=0&초록=', True)
red_str = my_values.get('빨강',[''])
green_str = my_values.get('초록',[''])

print(f'{red_str=} {green_str=}')

# python에서 3항 연산자는 아래와 같이 사용한다.
red = int(red_str[0]) if isinstance(red_str[0], str) else 0
print(red)

# 위와 같은 조건보다는 아래와 같이 도우미 함수를 작성하는 것이 더 명확하다.
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
        return int(found[0])
    return default

print(get_first_int(my_values, '빨강', 0))
```

**- Result**
```text
red_str=['5'] green_str=['']
5
5
```

<br>

### 06. 인덱스를 사용하는 대신 대입을 사용해 언패킹하라

**- Code**
```python
item = ('호박엿', '식혜')

# index를 사용해서 접근
first = item[0]
second = item[1]
print(f'index : {first=} {second=}')

# 언패킹
first, second = item
print(f'unpacking : {first=} {second=}')


# 실제 코드에 아래의 코드는 사용되면 안되나, 패킹은 이런식으로도 가능하다
favorite_snacks = {
    '짭짤한 과자' : ('프레즐', 100),
    '단 과자' : ('쿠키', 180),
    '채소' : ('당근', 20),
}

((type1, (name1, cals1)),
(type2, (name2, cals2)),
(type3, (name3, cals3))) = favorite_snacks.items()

print(f'my favorite food is {type1} with {name1}, {cals1}')
print(f'my favorite food is {type2} with {name2}, {cals2}')
print(f'my favorite food is {type3} with {name3}, {cals3}')

# 언패킹의 원리로 아래와 같이 맞바꾸는 것도 가능하다.
a = 1
b = 2

# 우항에 있는 값이 튜플화 되고 그 값이 그대로 좌항에 대입이되는 방식이다.
# 1. 우항 b,a -> tuple (1, 1) 
# 2. 좌항 a,b = tuple(1, 1) 
print(f'{a=}, {b=}')
a,b = b,a
print(f'{a=}, {b=}')
```

**- Result**
```text
index : first='호박엿' second='식혜'
unpacking : first='호박엿' second='식혜'
my favorite food is 짭짤한 과자 with 프레즐, 100
my favorite food is 단 과자 with 쿠키, 180
my favorite food is 채소 with 당근, 20
a=1, b=2
a=2, b=1
```

<br>

### 07. range 보다는 enumerate를 사용하라
- 이건 잘쓰고 있으니 pass

<br>

### 08. 여러 이터레이터에 대해 나란히 루프를 수행하려면 zip을 사용하라

**- Code**
```python
# example
names = ['Cecilia', '남궁민수', '川材']
counts = [ len(name) for idx, name in enumerate(names)]
print(names, counts)

# 가장 큰 길이를 가진 name을 구해보자 
# zip을 사용하지 않을 경우
def get_longest(names, counts):
    longest_name = None
    max_count = 0
    for i, name in enumerate(names):
        count = counts[i]
        if count > max_count:
            longest_name = name
            max_count = count
    return longest_name, max_count

# zip을 사용할 경우 : zip을 사용할 경우 더 간결하게 작성하는 것이 가능하다.
def get_longest_zip(names, counts):
    longest_name = None
    max_count = 0
    for name, count in zip(names, counts):
        if count > max_count:
            longest_name = name
            max_count = count
    return longest_name, max_count

print(f'{get_longest(names, counts)=}')
print(f'{get_longest_zip(names, counts)=}')

names.append('Rosalind')
# 이름을 추가 했으나 원하는 결과가 나오지 않는다.
# zip은 두 container 중 길이를 짧은 것을 기준으로 합친다.
# len(names) = 4 , len(counts) = 3 
print(f'{get_longest_zip(names, counts)=}')

# 길이가 긴 것을 기준으로 zip을 사용하고 싶다면 itertools를 이용한다.
import itertools
zip_long = [(name, count) for name, count in itertools.zip_longest(names, counts)]
print(f'{zip_long=}')

```

**- Result**
```text
['Cecilia', '남궁민수', '川材'] [7, 4, 2]
get_longest(names, counts)=('Cecilia', 7)
get_longest_zip(names, counts)=('Cecilia', 7)
get_longest_zip(names, counts)=('Cecilia', 7)
zip_long=[('Cecilia', 7), ('남궁민수', 4), ('川材', 2), ('Rosalind', None)]
```

<br>

### 09. for나 while 루프 뒤에 else 블록을 사용하지 말라

- 파이썬은 for나 while 루프에 속한 블록 바로 뒤에 else 블록을 허용하는 특별한 문법을 제공한다.
- 루프 뒤에 오는 else 블록은 루프가 반복되는 도중에 break를 만나지 않은 경우에만 실행된다.
- 동작이 직관적이지 않고 혼동을 야기할 수 있으므로 루프 뒤에 else 블록을 사용하지 말라.

<br>

### 10. 대입식(:=)을 사용해 반복을 피하라

**- Code**
```python
fresh_fruits = {
    '사과' : 10,
    '바나나' : 8,
    '레몬' : 5
}

# 실제로는 LINE 13까지만 사용하는데, 그 이후에도 사용이 되는 것인지 명확하게 구분하기 힘들다.
counts = fresh_fruits.get('레몬', 0 ) 
if counts :
    print('lemon exists')
else :
    print('no lemon')

# := 대입식을 통해 명확하게 변수의 사용 위치를 표시할 수 있다.
if counts := fresh_fruits.get('레몬', 0 ) :
    print('lemon exists')
else :
    print('nothing')

# 다른 숫자를 비교할 경우 반드시 괄호를 포함해야 한다.
if ( counts := fresh_fruits.get('사과', 0 ) ) >= 4 :
    print('apple exceed 4.')
else :
    print(' < 4')

# python에서는 switch와 같은 구문이 없어서 비슷한 흉내를 내려면 아래와 같이 지저분한 코드가 나온다.
counts = fresh_fruits.get('사과', 0 )
if counts > 2 :
    print('사과 먹는다')
else : 
    counts = fresh_fruits.get('바나나', 0)
    if counts > 5 :
        print('바나나 먹자')
    else :
        counts = fresh_fruits.get('레몬',0)
        if counts > 3 :
            print('레몬 먹자')

# :=을 이용하면 깔끔하게 표현할 수 있다.
if ( counts := fresh_fruits.get('사과', 0)) > 2 :
    print('사과 먹자')
elif ( counts := fresh_fruits.get('바나나', 0)) > 5 :
    print('바나나 먹자')
elif ( counts := fresh_fruits.get('레몬', 0)) > 3 :
    print('레몬 먹자')


# while 문에서도 활용이 가능하다.
def pick_fruits():
    a = [1,2,3,None]
    yield from a

# while 조건문에 a를 넣기 위해 앞에서 next(num)을 한번 호출했으나 그렇기 보기좋지 않다.
num = pick_fruits()
a = next(num)
while a :
    print(a)
    a = next(num)

# 위의 사항을 보완하기 위해 "무한 루프 - 중간에서 끝내기" pattern을 사용했으나 여전히 깔끔하지는 않다.
num = pick_fruits()
while True :
    a = next(num)
    if not a :
        break
    print(a)
    
# := 을 사용하면 깔끔하게 코드를 작성할 수 있다.
num = pick_fruits()
while a := next(num):
    print(a)

```

**- Result**
```text
lemon exists
lemon exists
apple exceed 4.
사과 먹는다
사과 먹자
1
2
3
1
2
3
1
2
3
```

<br>

### template

**- Code**
```python

```

**- Result**
```text

```

<br>
