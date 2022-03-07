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
