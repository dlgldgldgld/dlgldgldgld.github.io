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

```

**- Result**
```text

```

<br>

### 22. 변수 위치 인자를 사용해 시각적인 잡음을 줄여라

**- Code**
```python

```

**- Result**
```text

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
