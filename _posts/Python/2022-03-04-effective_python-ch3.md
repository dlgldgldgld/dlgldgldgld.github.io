---
layout: single
title:  "[파이썬 코딩의 기술 : Effective PYTHON 2ND 요약 및 코드 정리] CHAPTER 3. 함수"
category: Python
tag: Python
---

## Chapter 3. 함수 

### 19. 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹 하자 말라
아래 코드를 예시로 보면, 언패킹 값이 너무 많아졌을때 발생하는 문제들에 대해서 알 수 있다.
그래서 4개 이상의 언패킹은 사용하지 않는 것을 권장한다. ( 3개까지만 허용한다는 뜻 )
이보다 더 많은 값을 언패킹하고 싶다면 경량 클래스나 [namedtuple](https://docs.python.org/ko/3.7/library/collections.html#collections.namedtuple)을 사용하는게 더 낫다.

**- Code**
```python
# 몸길이(lengths)의 최소, 최대, 평균, 중앙값(median), 개체수에 알아보고 싶다고 가정하자.
# 아래의 함수를 구현할 수 있다.
def get_stats(numbers):
    minimum = min(numbers)
    maximum = max(numbers)
    count = len(numbers)
    average = sum(numbers)/count
    sorted_numbers = sorted(numbers)
    middle = count // 2
    if count % 2 == 0:
        lower = sorted_numbers[middle-1]
        upper = sorted_numbers[middle]
        median = (lower + upper) / 2
    else:
        median = sorted_numbers[middle]
    
    return minimum, maximum, average, median, count

lengths = [63,73,72,50,56,55,71,61,72,70]
minimum, maximum, average, median, count = get_stats(lengths)

# 위의 함수는 두가지 문제를 가진다.
# 1. 반환값이 너무 많아서 return 값을 받아올때 실수할 수도 있다.
minimum, maximum, average, median, count = get_stats(lengths) # ( O )
minimum, maximum, median, average, count = get_stats(lengths) # ( X )

# 2. 언패킹하는 부분이 너무 길고, 여러가지 방식으로 줄이 바뀔 수 있어서 가독성이 나빠진다.
# 1)
minimum, maximum, average, median, count = get_stats(
    lengths)

# 2)
(minimum, maximum, average, 
median, count) = get_stats(lengths)

# 3)
minimum, maximum, average, median, count = \
    get_stats(lengths)

# 4)
(minimum, maximum, average, median, count
) = get_stats(lengths)

```

<br>

### 20. None을 반환하기보다는 예외를 발생시켜라

**- Code**
```python
# 아래의 코드는 a에 0이 들어갔을때도 return ( 0 = None )이 되는데, 
# if 절에 True가 아닌지 검사를 한다면 의도치 않은 동작을 야기시킬 수 있다.
# 아래 예제를 보자.
def careful_divide(a,b):
    try : 
        return a / b
    except ZeroDivisionError :
        return None

x,y = 1,0
result = careful_divide(x,y)
if not result :
    print ('#1. Division 에러')

x,y = 0,5
result = careful_divide(x,y)
if not result :
    print ('#2. Division 에러')


# 이를 대비하는 방안은 tuple을 return해서 기준 값을 늘려서 오류를 체크하거나, 아예 에러를 발생시켜 버리는 것으로 대응이 가능하다.
def careful_divide(a,b):
    try : 
        return a / b
    except ZeroDivisionError as e:
        raise ValueError('잘못된 입력')

x, y = 5, 2
try :
    result = careful_divide(x,y)
except ValueError :
    print('잘못된 입력')
else :
    print('결과는 %.1f 입니다.' % result)

# 확장해서 어노테이션을 사용해 None이 아니라고 명시하거나 Docstring에 설명을 추가하면 좋다.
# 이렇게 하면 개발자는 잘못 사용할 확률이 훨씬 줄어든다. = 잘못쓴사람 탓을 하는 것이 가능하다
def careful_divide(a,b) -> float :
    """a를 b로 나눈다.

    Raises:
        ValueError: b가 0이어서 나눗셈을 할 수 없을 때
    """
    try : 
        return a / b
    except ZeroDivisionError as e:
        raise ValueError('잘못된 입력')

x,y = 5,0
result = careful_divide(x,y)
if not result :
    print ('#2. Division 에러')
```

**- Result**
```text
#1. Division 에러
#2. Division 에러
결과는 2.5 입니다.
Traceback (most recent call last):
  File "d:\test\hello.py", line 45, in careful_divide
    return a / b
ZeroDivisionError: division by zero

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "d:\test\hello.py", line 50, in <module>
    result = careful_divide(x,y)
  File "d:\test\hello.py", line 47, in careful_divide
    raise ValueError('잘못된 입력')
ValueError: 잘못된 입력
```

<br>

### 21. 변수 영역과 클로저의 상호작용 방식을 이해하라

**- Code**
```python
import dis
# 정렬을 하는데 group 안에 있는 것들이 앞으로 올 수 있도록 정렬을 하고싶다고 하자.
# 이를 구현하기 위해서는 아래와 같은 함수가 나올 수 있다.
def sort_priority1(values, group):
    def helper(x):
        if x in group:
            return (0,x)
        return (1,x)
    values.sort(key = helper)

# 근데 values안에 group이 포함되었는지 안되었는지에 대해서도 알고싶다고 하자.
# 그러니깐 return (0,x)가 한번이라도 나왔다면 true를 호출하고 싶다고 하자.
# 그때 아래와 같이 손쉽게 구현이 가능하다고 생각할 수 있다.
# 하지만 아래의 코드에서는 helper안의 found는 외부의 found가 아닌 새로운 found를 할당한다. 
# 왜냐면 지역함수안에서의 변수는 항상 새롭게 할당되는 Python의 특성 때문이다.
def sort_priority2(values, group):
    found = False
    def helper(x):
        if x in group:
            found = True
            return (0,x)
        return (1,x)
    values.sort(key = helper)
    return found

dis.dis(sort_priority2)

# 만약에 외부의 변수를 사용하고 싶다면 nonlocal을 통해 group과 같이 closure 취급하는게 가능하다.
def sort_priority2(values, group):
    found = False
    def helper(x):
        if x in group:
            nonlocal found
            found = True
            return (0,x)
        return (1,x)
    values.sort(key = helper)
    return found

print("---------------------------------------------------------------------------")
dis.dis(sort_priority2)
```

**- Result**
```text
 17           0 LOAD_CONST               1 (False)
              2 STORE_FAST               2 (found)

 18           4 LOAD_CLOSURE             0 (group)
              6 BUILD_TUPLE              1
              8 LOAD_CONST               2 (<code object helper at 0x00000226146767C0, file "d:\test\hello.py", line 18>)

......

Disassembly of <code object helper at 0x00000226146767C0, file "d:\test\hello.py", line 18>:
 19           0 LOAD_FAST                0 (x)
              2 LOAD_DEREF               0 (group)
              4 CONTAINS_OP              0
              6 POP_JUMP_IF_FALSE       20

 20           8 LOAD_CONST               1 (True)
             10 STORE_FAST               1 (found)

......

---------------------------------------------------------------------------
 30           0 LOAD_CONST               1 (False)
              2 STORE_DEREF              0 (found)

 31           4 LOAD_CLOSURE             0 (found)
              6 LOAD_CLOSURE             1 (group)
              8 BUILD_TUPLE              2
             10 LOAD_CONST               2 (<code object helper at 0x0000022614676920, file "d:\test\hello.py", line 31>)

......

Disassembly of <code object helper at 0x0000022614676920, file "d:\test\hello.py", line 31>:
 32           0 LOAD_FAST                0 (x)
              2 LOAD_DEREF               1 (group)
              4 CONTAINS_OP              0
              6 POP_JUMP_IF_FALSE       20

 34           8 LOAD_CONST               1 (True)
             10 STORE_DEREF              0 (found)

```

<br>

### 22. 변수(가변) 위치 인자를 사용해 시각적인 잡음을 줄여라
위의 제목에서 변수는 가변을 뜻하는 것 같다. 오타인 것으로 보인다.
여기서 가변은 asterisk(*) 을 뜻한다.

주의 사항에 나온 것들을 조심해서 가변인자를 사용하라고 권장한다. 
근데 두번째는 솔직히 억지같다.

**- Code**
```python
def log( message, *values) :
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('내 숫자는', 1, 2)
log('안녕')

# 가변인자 사용시 주의해야할 것이 2가지 있다. 
# 이를 주의해서 쓰자.

# 가변인자의 문제 1. Generator를 인자로 받는 경우 tuple 화 된다.
def my_generator():
    for i in range(10):
        yield i

def my_func(*args):
    print(args)

it = my_generator()
my_func(*it)

# 가변인자의 문제 2. 인자에 변수를 잘못쓸 경우 분석이 힘듬
def log(seq, msg, *val):
    if not val:
        print(f'{seq}-{msg}')
    else:
        val_str = ','.join(str(x) for x in val)
        print(f'{seq}-{msg}:{val_str}')

log(1, '좋아하는 숫자는?', 7, 33)
log(1, '안녕')
log('좋아하는 숫자는?', 7, 33) # 문제 발생
```

**- Result**
```text
내 숫자는: 1, 2
안녕
(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
1-좋아하는 숫자는?:7,33
1-안녕
좋아하는 숫자는?-7:33
```

<br>

### 23. 키워드 인자로 선택적인 기능을 제공하라

**- Code**
```python
# 우선 키워드 인자의 예시부터 알아보자

def remainder( number, divisor ) :
    return number % divisor

assert remainder(20, 7) == 6
# 아래 예시들을 보면 args에 keyword와 함께 값을 대입하는 모습을 볼 수 있는데
# 이를 키워드 인자를 사용한 function 이라고 볼 수 있다.
print(f'{remainder(20, 7)=}')
print(f'{remainder(20, divisor=7)=}')
print(f'{remainder(number = 20, divisor = 7)=}')
print(f'{remainder(divisor = 7, number = 20)=}')

# 키워드 인자는 무조건 위치 args 보다 뒤에 정의되어야 한다. 그렇지 않으면 에러가 발생한다.
# remainder( number=20, 7 )

# ** 연산자를 통해 map을 unpacking 해서 인자로 전달하는 것이 가능하다.

my_kwargs = {
    'number' : 20,
    'divisor' : 7,
}

print (f'{remainder(**my_kwargs)=}')

# 키워드 인자 사용시 발생하는 장점들
# 1. 코드를 처음보는 사람들에게 함수 호출의 의미를 명확하게 알려줄 수 있음.
#    그냥 인자를 툭 던져주는 것보다는 "나머지를 구하는데 필요한게 number와 divisor가 있다."" 라고 명확하게 의미 파악이 가능.
print(f'{remainder(20, 7)=}')
print(f'{remainder(number = 20, divisor = 7)=}')

# 2. 키워드 인자의 경우 함수 정의에서 디폴트 값을 지정할 수 있다.
#    ( 이거는 상관없는 얘기 아닌가 )
def flow_rate(weight_diff, time_diff, period = 1):
    return (weight_diff / time_diff ) * period
    
# 3. 기존 사용자에게 호환성을 제공하며 function 변경이 가능.
#    아래와 같이 default 값을 미리 설정하면 기존 의도를 해치지 않은 상태로 소스 변경을 하는 것이 가능하다.

# 수정 전
def flow_rate(weight_diff, time_diff):
    return (weight_diff / time_diff )

# 수정 후
def flow_rate(weight_diff, time_diff, period = 1):
    return (weight_diff / time_diff ) * period

flow_rate(2,3, period=3.3)
```

**- Result**
```text
remainder(20, 7)=6
remainder(20, divisor=7)=6
remainder(number = 20, divisor = 7)=6
remainder(divisor = 7, number = 20)=6
remainder(**my_kwargs)=6
remainder(20, 7)=6
remainder(number = 20, divisor = 7)=6
```

### 24. None과 독스트링을 사용해 동적인 디폴트 인자를 지정하라

**- Code**
```python
# default 인자의 잘못된 사용방법
# 아래 코드에서 when이 같은 이유는 "default 인자"는 module 호출시 최초의 실행된 값을 기준으로만 할당한다.
# now()함수는 호출할때마다 실행되는 개념이 아니고, 한번만 실행 된 후 어딘가에 저장된 이후에는 저장된 값을 빼쓰는 구조라는 뜻이다.

import datetime
from time import sleep
def log(msg, when = datetime.datetime.now()):
    print(f'{when}-{msg}')

log("안녕!")
sleep(0.1)
log("또 안녕!")


# 위의 경우에는 원하는 동작을 하기 위해 디폴트 인자로 None을 지정하고 실제 동작을 독스트링에 기재하는 방법이 있다.
def log(msg, when=None):
    """메시지와 타임스태프를 로그에 남긴다.

    Args:
        message: 출력할 메시지.
        when : 메시지 발생한 시각(datetime).
               디폴트 값은 현재 시간이다.
    """

    if when is None :
        when = datetime.datetime.now()
    print(f'{when}:{msg}')

log("안녕!")
sleep(0.1)
log("또 안녕!")

# 위와 관련된 또다른 오류 예시다.
# foo와 bar는 decode의 default를 공유하기 때문에 동일한 주소를 가진 값으로 인식한다.

import json
def decode(data, default={}):
    try:
        return json.loads(data)
    except ValueError:
        return default

foo = decode('잘못된 데이터')
foo['stuff'] = 5
bar = decode('잘못된 데이터')
bar['meep'] = 1
print('Foo:', foo)
print('Bar:', bar)

assert foo is bar

# 의도에 맞게 동작하도록 decode 함수를 None과 독스트링을 사용해 개선해보자.
def decode(data, default=None):
    """문자열로부터 JSON 데이터를 읽어온다.

    Args:
        ~~
    """
    try:
        return json.loads(data)
    except ValueError:
        if default is None:
            default = {}
        return default

foo = decode('잘못된 데이터')
foo['stuff'] = 5
bar = decode('잘못된 데이터')
bar['meep'] = 1
print('Foo:', foo)
print('Bar:', bar)
assert foo is not bar

```

**- Result**
```text
2022-03-06 10:34:10.309829-안녕!
2022-03-06 10:34:10.309829-또 안녕!
2022-03-06 10:34:10.412265:안녕!
2022-03-06 10:34:10.523557:또 안녕!
Foo: {'stuff': 5, 'meep': 1}
Bar: {'stuff': 5, 'meep': 1}
Foo: {'stuff': 5}
Bar: {'meep': 1}
```

### 25. 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들어라

**- Code**
```python
# Default 인자 및 위치적 인자를 강제로 적용하기 싶은 경우가 있다.
# 예를 들면 아래의 예시에서는 numerator, denominator의 경우에는 argument의 name이 자주 바뀔 수 있어
# 위치적 인자로만 설정하고 싶고, ignore_overflow, ignore_zero_division은 사용자가 의미를 명확하게 사용할 수 있도록
# keyword 인자로만 사용하게 싶은 경우이다.
# 이럴 경우 parameter에 `/`나 `*`을 적절히 활용하면 의도한 바를 명확하게 지정할 수 있다.
# 1. `/` : 위치적 인자로 지정하고 싶은 곳까지의 바로 뒤에 /를 쓰면 그 값들은 keyword 인자로 사용되는 것이 불가능하다.
#          그렇기 때문에 positional argument로만 사용되도록 강제된다.
# 2. `*` : keyword 인자로만 사용하고 싶은 곳 앞에 붙인다. 그러면 어떤 index에 keyword 인자가 존재하는지 체크가 불가능하기 때문에
#          keyword 인자로만 사용되도록 강제된다.
# 3. '/' ~ '*' : 여기에 들어가는 값은 positional, keyword로 모두 적용가능한 인자가 된다.
def safe_division_e(numerator, denominator, /,
                    ndigits=10, *,
                    ignore_overflow=False,
                    ignore_zero_division=False):
    try:
        fraction = numerator / denominator
        return round(fraction, ndigits)
    except OverflowError:
        if ignore_overflow:
            return 0
        else :
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

print(f'{safe_division_e(22,7)=}')
print(f'{safe_division_e(22,7,5)=}')
print(f'{safe_division_e(22,7,ndigits=2)=}')

```

**- Result**
```text
safe_division_e(22,7)=3.1428571429
safe_division_e(22,7,5)=3.14286
safe_division_e(22,7,ndigits=2)=3.14
```

### 26. functools.wrap을 사용해 함수 데코레이터를 정의하라

**- Code**
```python
# debug용으로 아래와 같은 함수를 작성해서 decorator로 넣어주곤 한다.
def trace(func):
    def wrapper(*args, **kwargs):
        res = func(*args, **kwargs)
        print(f'{func.__name__}({args!r}, {kwargs!r} '
              f'-> {res!r}')
        return res
    return wrapper

@trace
def fibonacci(n):
    if n in (0,1):
        return n
    return (fibonacci(n-2) + fibonacci(n-1))

# 잘 동작한다.
fibonacci(4)

# 허나 그냥 print를 날려보면 fibonacci 함수는 fibonacci가 아닌 wrapper 함수라고 정의되고 있음을 알수 있다.
print(fibonacci)
help(fibonacci)
# 이런 동작은 디버깅을 하거나 인트로스펙션(실행시점 프로그램이 어떻게 실행되는지 관찰) 할때 문제가 된다.
# 이런 경우에는 functools 내장 모듈에 있는 wraps 함수를 사용하는 방법이 있다.

from functools import wraps
def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        res = func(*args, **kwargs)
        print(f'{func.__name__}({args!r}, {kwargs!r} '
              f'-> {res!r}')
        return res
    return wrapper

@trace
def fibonacci(n):
    """
        n번째 피보나치를 반환하는 함수인데요.
    """
    if n in (0,1):
        return n
    return (fibonacci(n-2) + fibonacci(n-1))

#그러면 의도와 같이 정확하게 나옴을 알 수 있다.
print(fibonacci)
help(fibonacci)
```

**- Result**
```text
fibonacci((0,), {} -> 0
fibonacci((1,), {} -> 1
fibonacci((2,), {} -> 1
fibonacci((1,), {} -> 1
fibonacci((0,), {} -> 0
fibonacci((1,), {} -> 1
fibonacci((2,), {} -> 1
fibonacci((3,), {} -> 2
fibonacci((4,), {} -> 3
<function trace.<locals>.wrapper at 0x00000289683FCD30>
Help on function wrapper in module __main__:

wrapper(*args, **kwargs)

<function fibonacci at 0x0000028968A0AD30>
Help on function fibonacci in module __main__:

fibonacci(n)
    n번째 피보나치를 반환하는 함수인데요.
```
