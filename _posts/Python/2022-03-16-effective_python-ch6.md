---
layout: single
title:  "[파이썬 코딩의 기술 : Effective PYTHON 2ND 요약 및 코드 정리] CHAPTER 6. 메타클래스와 애트리튜브"
category: Python
tag: Python
---

# Meta Class란?
Python에서의 Meta-class는 class를 가로채서 거기에 특별한 동작을 제공하는 클래스를 뜻한다. 예를들면 list 자료형에 replace 라는 기능을 추가하고 싶다고 가정하자. 여기서 replace 기능은 string의 replace 기능을 뜻한다. 즉 입력받은 항목을 다음 입력받은 항목으로 교체하는 기능을 뜻한다.

이러한 기능을 구현하기 위해 type 함수를 사용해서 간단하게 메타클래스를 정의할 수 있다.

```python
def replace(self, old, new):
    while old in self:
        self[self.index(old)] = new

AdvList = type('AdvancedList', (list, ), { 'replace' : replace })
```

이처럼 클래스문을 가로채서 다른 임시 클래스를 만드는 것을 메타클래스라고 하는데 이와 attribute에 대한 best-practice에 대해서 살펴보자.

### 44. 세터와 게터 메서드 대신 평범한 애트리뷰트를 사용하라
Python에서는 getter와 setter를 직접 구현해야 한다. 하지만 이미 이보다 더 좋은 property 라는 기능을 이미 알고있다. 

property를 통해 final 한정자와 비슷한 효과를 줄 수 있는 등, 값의 범위 설정과 같은 다양한 구현을 추가할 수 있다.

get,set 구현과 같은 간단한 것이 있다면 프로퍼티를 사용하라.

```python
class FixedResistance():
    def __init__(self, ohms):
        pass
    
    @property
    def ohms(self):
        return self._ohms

    # example 1. 불변 객체
    @ohms.setter
    def ohms(self, ohms):
        if hasattr(self, '_ohms'):
            raise AttributeError("Ohms는 불변 객체입니다")
        self._ohms = ohms

    # example 2. 값 범위 한정
    @ohms.setter
    def ohms(self, ohms):
        if ohms <= 0:
            raise ValueError("저항 > 0 이어야 함.")
        self._ohms = ohms
```

<br>



### 45. 애트리뷰트를 리팩터링하는 대신 @property를 사용하라

@property를 사용해 기존 인스턴스 애트리뷰트에 새로운 기능을 제공할 수 있다.
리팩터링 하기전 간단한 것이라면 @property를 써라는 의미로 해석된다.

단, 너무 과하게 사용하고 있다면, 클래스와 클래스를 사용하는 모든 코드를 리팩터링 할 것을 권장하고 있다.

<br>

### 46. 재사용 가능한 @property 메서드를 만들려면 디스크립터를 사용하라

디스크립터에 관한 설명으로 [https://dlgldgldgld.github.io/python/descriptor/](https://dlgldgldgld.github.io/python/descriptor/) 로 대체.

<br>

### 47. 지연 계산 애트리뷰트가 필요하면 `__getattr__`, `__getattribute__`, `__setattr__` 을 사용하라
<br>

**함수 설명**

|method|설명|
|----|----|
|`__getattr__`|값에 접근할때 호출. 단, 이미 __dict__에 변수가 있을때는 호출되지 않음.|
|`__getattribute__`|값에 접근할때 호출. __dict__에 있든 없든 일단 호출|
|`__setattr__`|인스턴스 변수로 삽입할때 호출. 값이 있든 없든 모두 접근함.|

**주의 사항**

`__getattribute__` , `__setattr__` 에서 `self.변수`의 형태로 값에 접근을 하면 또다시 `__getattribute__`나 `__setattr__`에 접근하기 때문에 무한 루프에 빠지게 된다. 때문에 이런 상황에서는 `super()`를 사용해서 값에 접근해야한다.
{: .notice--warning}


**- Code**
```python
class LazyRecord:
    def __init__(self):
        self.exists = 5

    def __getattr__(self, name):
        value = f'{name}를 위한 값'
        setattr(self, name, value)
        return value

class LoggingLazyRecord(LazyRecord):
    def __getattr__(self, name):
        print(f'* 호출: __getattr__({name!r}), ' f'인스턴스 딕셔너리 채워넣음')
        result = super().__getattr__(name)
        print(f'* 반환: {result!r}')
        return result

class ValidatingRecord:
    def __init__(self):
        self.exists = 5

    def __getattribute__(self, name):
        print(f'* 호출: __getattribute__({name!r})')
        try:
            value = super().__getattribute__(name)
            print(f'* {name!r} 찾음, {value!r} 반환')
            return value
        except AttributeError:
            value = f'{name}를 위한 값'
            print(f'* {name!r}를 {value!r}로 설정')
            setattr(self, name, value)
            return value


print('------------------------------------')
print('LoggingLazyRecord Test')
data = LoggingLazyRecord()
print('이전:', data.__dict__)
print('최초에 foo가 있나:', hasattr(data, 'foo'))
print('이후:', data.__dict__)
print('이후에 foo가 있나:', hasattr(data, 'foo'))
print('------------------------------------')
print('------------------------------------')
print('ValidatingRecord Test')
data = ValidatingRecord()
print('최초에 foo가 있나:', hasattr(data, 'foo'))
print('이후에 foo가 있나:', hasattr(data, 'foo'))
# foo의 값에 접근만해도 이렇게 호출을 해버림
print(data.foo)

class SavingRecord:
    def __setattr__(self, name, value):
        super().__setattr__(name, value)

class LoggingSavingRecord(SavingRecord):
    def __setattr__(self, name, value):
        print(f'* 호출: __setattr__({name!r}, {value!r})')
        super().__setattr__(name, value)

print('------------------------------------')
print('------------------------------------')
print('LoggingSavingRecord Test')
data = LoggingSavingRecord()
print('이전:', data.__dict__)
data.foo = 5
print('이후:', data.__dict__)
data.foo = 7
print('최후:', data.__dict__)

```

**- Result**
```text
LoggingLazyRecord Test
이전: {'exists': 5}
* 호출: __getattr__('foo'), 인스턴스 딕셔너리 채워넣음
* 반환: 'foo를 위한 값'
최초에 foo가 있나: True
이후: {'exists': 5, 'foo': 'foo를 위한 값'}
이후에 foo가 있나: True
------------------------------------
------------------------------------
ValidatingRecord Test
* 호출: __getattribute__('foo')
* 'foo'를 'foo를 위한 값'로 설정
최초에 foo가 있나: True
* 호출: __getattribute__('foo')
* 'foo' 찾음, 'foo를 위한 값' 반환
이후에 foo가 있나: True
* 호출: __getattribute__('foo')
* 'foo' 찾음, 'foo를 위한 값' 반환
foo를 위한 값
------------------------------------
------------------------------------
LoggingSavingRecord Test
이전: {}
* 호출: __setattr__('foo', 5)
이후: {'foo': 5}
* 호출: __setattr__('foo', 7)
최후: {'foo': 7}
```

<br>

### 48. `__init_subclass__`를 사용해 하위 클래스를 검증하라

`__init_subclass__` 함수를 사용하지 않고 하위 클래스를 검증하는 방법은 metaclass를 사용하는 것이다.

```python
class ValidatePolygon(type):
    def __new__(meta, name, bases, class_dict):
        # Polygon 클래스의 하위 클래스만 검증한다
        if bases:
            if class_dict['sides'] < 3:
                raise ValueError('다각형 변은 3개 이상 있어야 함')
        return type.__new__(meta, name, bases, class_dict)

class Polygon(metaclass = ValidatePolygon):
    sides = None
    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Triangle(Polygon):
    sides = 3

class Rectangle(Polygon):
    sides = 4

class Nonagon(Polygon):
    sides = 9

assert Triangle.interior_angles() == 180
assert Rectangle.interior_angles() == 360
assert Nonagon.interior_angles() == 1260
```

이 방법은 여러가지 단점이 있는데 첫째로 코드가 복잡해 보이는 것, 두번째는 메타클래스를 여러개 사용하려 하기가 까다로움 등이 있다.

`__init_subclass__` 는 이러한 불편한 점을 해소하기 위해 Python 3.6 부터 지원되는 magic method 이다.
이를 사용하면 위의 코드는 다음과 같이 간편하게 구현할 수 있다.

```python
class BetterPolygon():
    sides = None

    def __init_subclass__(cls) -> None:
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError('다각형 변은 3개 이상이어야 함')

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Hexagon(BetterPolygon):
    sides = 6

assert Hexagon.interior_angles() == 720
```

위에서 언급된 여러개의 class를 상속받는 것도 가능하다.

```python
class Polygon():
    sides = None

    def __init_subclass__(cls) -> None:
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError('다각형 변은 3개 이상이어야 함')

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Filled():
    color = None

    def __init_subclass__(cls)-> None:
        super().__init_subclass__()
        if cls.color not in ('red', 'green', 'blue'):
            raise ValueError('지원하지 않는 color 값')


class RedTriangle(Filled, Polygon) :
    color = 'red'
    sides = 3

rubby = RedTriangle()
assert isinstance(rubby, Filled)
assert isinstance(rubby, Polygon)
```

<br>


### xx. Template

**- Code**
```python

```

**- Result**
```text

```

<br>