---
layout: single
title:  "[파이썬 코딩의 기술 : Effective PYTHON 2ND 요약 및 코드 정리] CHAPTER 4. 컴프리헨션과 제너레이터"
category: Python
tag: Python
---

## Chapter 4. 컴프리헨션과 제너레이터 

### 27. map과 filter 대신 컴프리헨션을 사용하라

**- Code**
```python
a = [1,2,3,4,5,6,7,8,9,10]

# map을 comprehension을 사용해 더 가독성이 좋게 표현할 수 있다.
alt = list(map(lambda x: x**2, a))
print(f'{alt=}')
alt = [ i**2 for i in a ]
print(f'{alt=}')

# 이는 filter 함수또한 마찬가지이다.
alt = list(filter(lambda x : x % 2 == 0, a))
print(f'{alt=}')
alt = [ i for i in a if i % 2 == 0 ]
print(f'{alt=}')

# 그리고 컴프리헨션에서는 아래와 같이 Dict과 Set에 대해서도 적용이 가능하다.
# 이것들도 본래 함수를 사용하는 것과 비교하면 훨씬 깔끔하다.
alt = set(map(lambda x : x ** 3, filter (lambda x : x % 2 == 0, a )))
print(f'{alt=}')
alt = { i**3 for i in a if i % 2 == 0 }
print(f'{alt=}')

alt = dict(map(lambda x : (x, x**2), filter(lambda x : x % 2 == 0 , a)))
print(f'{alt=}')
alt = { i : i**2 for i in a if i % 2 == 0 }
print(f'{alt=}')
```

**- Result**
```text
alt=[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
alt=[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
alt=[2, 4, 6, 8, 10]
alt=[2, 4, 6, 8, 10]
alt={512, 64, 8, 1000, 216}
alt={512, 64, 8, 1000, 216}
alt={2: 4, 4: 16, 6: 36, 8: 64, 10: 100}
alt={2: 4, 4: 16, 6: 36, 8: 64, 10: 100}
```

<br>

### 28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라

**- Code**
```python
matrix = [ [1,2,3], [4,5,6], [7,8,9]]
# 쭉 나열하면 left -> right 순으로 for문이 형성
flat = [ x for row in matrix for x in row ]
print(f'{flat=}')
# 괄호가 달리면 right -> left 순으로 for문이 형성
squared = [ [x for x in row] for row in matrix ]
print(f'{squared=}')

# for loop가 3번 중첩될때부터는 소스가 개판이된다. 그냥 for문을 사용하자
my_list=[[[1,2,3],[4,5,6]],]
flat = [x for sublist1 in my_list for sublist2 in sublist1 for x in sublist2 ]
print(f'{flat=}')

flat = []
for sublist1 in my_list :
    for sublist2 in sublist1 :
        flat.extend(sublist2)

# 그리고 comprehension은 조건문을 여러개 사용하는 것이 가능하다
a = [i for i in range(1,11)]
b = [x for x in a if x > 4 if x % 2 == 0]
c = [x for x in a if x > 4 and x % 2 == 0]

print(f'{a=}, {b=}, {c=}')
```

**- Result**
```text
flat=[1, 2, 3, 4, 5, 6, 7, 8, 9]
squared=[[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat=[1, 2, 3, 4, 5, 6]
a=[1, 2, 3, 4, 5, 6, 7, 8, 9, 10], b=[6, 8, 10], c=[6, 8, 10]
```

<br>


### 29. 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라

**- Code**
```python
# 예제로 조립 제품들을 팔려고 할때 그 개수만큼 존재하는지 확인을 하고싶다고 하자.
# 아래의 못을 8개 구매하고 싶은데 재고가 그만큼 만족하는지 찾고싶다는 뜻이다.
stock = {
    '못' : 125,
    '나사못': 35,
    '나비너트' : 8, 
    '와셔' : 24,
}

order = ['나사못', '나비너트', '클립']

def get_batches(count, size):
    return count // size 

result = {}
# 일반 for문으로 구현하면 아래와 같이 나온다.
for name in order:
    count = stock.get(name, 0)
    # 요구 수량만큼 개수가 있다면 batches의 몫은 1 이상일 것이므로 아래와 같이 판단한다
    batches = get_batches(count, 8)
    if batches:
        result[name] = batches

# 여기서 위험할 수 있는 부분은 for문 안의 name 변수가 밖으로 노출이 가능하다는 것이고,
# 일단 소스도 조금 길어보이는게 없지않아 있다.
print(f'{name=}')

# 이를 개선하기 위해 컴프리헤이션으로 나타내보면 다음과 같다.
# 여기서 문제는 get_batches와 stock.get을 두번 호출한 것. 그래서 반복작업이 생긴다.
found = {name:get_batches(stock.get(name, 0),8)
         for name in order if get_batches(stock.get(name,0),8)}
print(f'{found=}')

# 이것을 왈러스 연산자를 사용하면 보다 더 깔끔하게 표시할 수 있다.
# 예제에서는 컴프리헨션의 루프 변수 누출이 되지 않는다고 되어있으나 실제로는 난다.
found = {(name, psb) for name in order 
          if ( psb := get_batches(stock.get(name,0),8 ) ) }
print(f'{found=}')


```

**- Result**
```text
name='클립'
found={'나사못': 4, '나비너트': 1}
found={('나비너트', 1), ('나사못', 4)}
```

<br>

### 30. 리스트를 반환하기보다는 제너레이터를 사용하라

**- Code**
```python
# 예시를 위해 텍스트 안에 특정 문자의 index를 찾고픈 기능을 구현하고 싶다고 하자.
# 나는 잘 공감이 안되지만 아래의 코드에서는 result.append 구문 자체가 너무 길어서 (index + 1)이 강조가
# 잘 안되는 문제점이 있다고 한다. 그래서 result.append 구문말고 yield를 사용해서 generator object를 만들어보자
def index_words(text, word = ' '):
    result = []
    if text : 
        result.append(0)
    for index, letter in enumerate(text):
        if letter == word:
            result.append(index + 1)
    return result

# 위의 코드와 비교해서 좀 더 꺨끔하게 나타남을 알수있다.
# 이렇게 사용하면 메모리 부담을 줄일 수 있기 상황에 따라서 좋다. 
# ( 겁나 큰 데이터를 메모리에 올리고 싶지않고 순회해서 하나하나씩 처리 하고싶은 경우)
def index_words_iter(text, word = ' '):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == word:
            yield index + 1

a = index_words_iter("hello", 'h')
for i in a :
    print(i)
    
# list화 시키는 것도 간단하다
a = list(index_words_iter("hello", 'h'))
print(a[:10])
```

**- Result**
```text
0
1
[0, 1]
```

<br>

### 31. 인자에 대해 이터레이션할 때는 방어적이 돼라

**- Code**
```python
from functools import total_ordering


def normalize(numbers):
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

def read_visits(data_path):
    with open(data_path) as f:
        for line in f :
            yield int(line)

# 해당 코드는 아무것도 표시되지 않는다.
# 그 이유는 normalize 내부에서 sum에서 iterator를 모두 호출하고,
# 아래의 value를 순회하는 부분에서는 StopIterator가 호출되기 때문이다. 이는 별도의 에러로 잡히지도 않는다.
it = read_visits('sample.txt')
percentages = normalize(it)
print(f'percentages = normalize(it) - {percentages=}')

# 이를 위해서는 매번 제너레이터를 호출해서 접근을 해야한다.
def normalize_func(get_iter):
    total = sum(get_iter())
    result = []
    for value in get_iter():
        percent = 100 * value / total
        result.append(percent)
    return result

# 하지만 이렇게 람다를 사용하는 방식은 작성하기 귀찮다.
percentages = normalize_func(lambda : read_visits('sample.txt'))
print(f'normalize_func(lambda : read_visits()) - {percentages=}')
# 
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

visits = ReadVisits('sample.txt')
percentages = normalize(visits)
print(f'normalize(visits) - {percentages=}')

# Iterator 객체들을 거르고 싶다면 아래와 같이 collections.abc 에서 Iterator를 찾아올 수 있다.
from collections.abc import Iterator

def normalize_defensive(numbers):
    if isinstance(numbers, Iterator):
        print(f'{numbers=}')
        raise TypeError('컨테이너를 제공해야 합니다')
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = ReadVisits('sample.txt')
percentages = normalize_defensive(visits)
print(f'normalize_defensive(visits) - {percentages=}')
percentages = normalize_defensive([15, 35, 80])
print(f'normalize_defensive([15, 35, 80]) - {percentages=}')
percentages = normalize_defensive(read_visits('sample.txt'))
```

**- Result**
```text
percentages = normalize(it) - percentages=[]
normalize_func(lambda : read_visits()) - percentages=[11.538461538461538, 26.923076923076923, 61.53846153846154]
normalize(visits) - percentages=[11.538461538461538, 26.923076923076923, 61.53846153846154]
normalize_defensive(visits) - percentages=[11.538461538461538, 26.923076923076923, 61.53846153846154]
normalize_defensive([15, 35, 80]) - percentages=[11.538461538461538, 26.923076923076923, 61.53846153846154]     
numbers=<generator object read_visits at 0x000002033860BD60>
Traceback (most recent call last):
  File "d:\01.Develop\02.PYTHON\99.tutorial\work\hello.py", line 69, in <module>
    percentages = normalize_defensive(read_visits('sample.txt'))
  File "d:\01.Develop\02.PYTHON\99.tutorial\work\hello.py", line 56, in normalize_defensive
    raise TypeError('컨테이너를 제공해야 합니다')
TypeError: 컨테이너를 제공해야 합니다
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
