---
layout: single
title:  "[PYTHON] - Descriptor"
category: Python
tag: Python
---

python에서는 @property를 통해 attribute를 좀 더 알차게 표현할 수 있다. 
예를 들어 maximum value를 구하는 class를 생성한다고 가정하면 아래와 같이 표현하게 될 것이다.

```python
class maxval:
    def __init__(self, val):
        self._val = val

    def addval(self, val):
        if self._val < val :
            self._val = val

    def getval(self):
        return self._val

var = maxval(5)
var.addval(10)
var.addval(9)

print(f'{var.getval()=}')
```

하지만 @property를 사용하면 이 같은 구현을 attribute를 호출하는 것만으로도 구현할 수 있다.

```python
class maxval:
    def __init__(self, val):
        self._val = val

    @property
    def val(self):
        return self._val

    @val.setter
    def val(self, val):
        if self._val < val :
            self._val = val

var = maxval(5)
var.val = 10
var.val = 9

print(f'{var.val=}')
```

이 @property에는 한가지 단점이 있는데 막상 저렇게 정의를 하고나서 재사용이 불가능하다는 것이다.

예를 들어 val 변수를 다른 클래스에도 가져다 쓰고 싶다고 하면 그건 방법이 없다. 그 클래스에도 @property로 val을 추가하는 방안밖에 없는 것이다.

그래서 임의의 필드를 재사용 가능하도록 하는 것이 @property를 구현할때 사용한 Descriptor 라는 개념인데 이에 대해서 알아보도록 하자.

# Descriptor
Descriptor 기본 정의는 어떤 객체의 attribute 조회, 저장 및 삭제를 사용자가 직접 정의할 수 있도록 해주는 class 이다.

예를 들자면 int나 float 같은 자료형은 해당 클래스가 제공해주는 기본 옵션으로만 값을 저장할 수 있다. 허나 사용자는 음수에 대해서는 허용을 하고 싶지 않던가, 조회의 기준을 python 객체가 아닌 sqlite 내부의 table에 대해서 하고 싶은 등 (이런 것을 ORM 이라고 한다.) 다양한 상황에 대한 attribute 정의를 내리고 싶을때가 있다.

이를 도와주는 것이 descripter 이다. 소스와 함께 살펴보도록 하자.

```python
import os

class DirectorySize:
    def __get__(self, obj, objtype=None):
        print('__get__ 호출')
        return len(os.listdir(obj.dirname))

class Directory:
    size = DirectorySize()              # Descriptor instance
    def __init__(self, dirname):
        self.dirname = dirname          # Regular instance attribute
```

위의 예제는 DirectorySize()라는 디스크립터를 사용해 size라는 인스턴스를 정의한 예제이다.

DirectorySize 클래스를 보면 `__get__(self, obj, objtype=None)` 으로 함수가 정의되어 있는데 Directory 클래스나 인스턴스에서 size를 호출할때 저 함수가 동작하게 되는 것이다.

```python
a = Directory('D:/')
print(a.size)

>> __get__ 호출
>> 16
```

`__get__` 함수의 인자는 3가지가 존재한다. self, obj, objtype 가 있는데 각각이 의미하는 바는 다음과 같다.

|args|desc|
|----|----|
|self|자기 자신(위의 예제에서는 DirectorySize class)|
|obj|호출한 객체|
|objtype|객체의 타입(위의 예제에서는 Directory class)|

get이 있으니 마찬가지로 값을 저장하기 위한 `__set__` 또한 존재한다.

```python
def __set__(self, obj, value):
        logging.info('Updating %r to %r', 'age', value)
        obj._age = value
```

이런식으로 사용되는데 value 값을 읽어서 객체를 update 하던가 self로 자기 자신의 값을 컨트롤을 할 수 있다.

위의 `__get__` 함수에서 value만 받아오는 것이 다른 점이다. 

`a.size = 5` 라고 하면 value에는 5가 들어가고 obj 에는 a instance가 들어가는 방식이다.

`__set__`을 사용할때는 주의할 점이 있는데, descripter는 class 변수로만 삽입을 할 수 있기에 서로 다른 객체에서도 값을 공유해 버린다는 특징이 있다.

```python
import os

class DirectorySize:
    def __get__(self, obj, objtype=None):
        return self._value

    def __set__(self, obj, value):
        self._value = value

class Directory:
    size = DirectorySize()              # Descriptor instance
    len  = DirectorySize()              # Descriptor instance
    def __init__(self, dirname):
        self.dirname = dirname          # Regular instance attribute

a = Directory('D:/')
b = Directory('D:/')

a.size = 5
a.len = 4

assert a.size == b.size
assert a.len == b.len
```

위의 소스를 보면 a의 값만 변경이 되었으나 b값 또한 같이 변경되어 assert를 통과하고 있다.

이것을 해결하기 위해서 instance 마냥 사용하고 싶다면 dict이나 weakref를 사용하거나 간단하게 obj를 통해 신규 instance 변수를 만드는 방식으로 사용을 한다.

개인적으로는 공식문서에 소개된 방식인 obj를 통해 instance 변수를 사용하는 것이 best-practice 이지 않을까 생각해본다.

```python
import logging

logging.basicConfig(level=logging.INFO)

class LoggedAccess:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = '_' + name

    def __get__(self, obj, objtype=None):
        value = getattr(obj, self.private_name)
        logging.info('Accessing %r giving %r', self.public_name, value)
        return value

    def __set__(self, obj, value):
        logging.info('Updating %r to %r', self.public_name, value)
        setattr(obj, self.private_name, value)

class Person:
    key = 'title'
    name = LoggedAccess()                # First descriptor instance
    age = LoggedAccess()                 # Second descriptor instance

    def __init__(self, name, age):
        self.name = name                 # Calls the first descriptor
        self.age = age                   # Calls the second descriptor
        self.key = 'Person'

    def birthday(self):
        self.age += 1

pete = Person('Peter P', 10)
kate = Person('Catherine C', 20)

print(vars(pete))

>> INFO:root:Updating 'name' to 'Peter P'
>> INFO:root:Updating 'age' to 10
>> INFO:root:Updating 'name' to 'Catherine C'       
>> INFO:root:Updating 'age' to 20
>> {'_name': 'Peter P', '_age': 10, 'key': 'Person'}
```

위의 코드를 보면 한가지 의문점이 들수 있다. Person 객체의 변수들을 자세히보면 class 변수와 instance 변수가 모두 일치한다. 

근데 이상한 것은 Descriptor로 선언된 것은 self.name에서 Descriptor를 호출하고 str로 선언된 key는 instnace 변수가 사용되고 있음을 알 수 있는데 이는 호출의 우선 순위가 있기 때문이다.

이는 [invocation-from-an-instance](https://docs.python.org/ko/3/howto/descriptor.html#invocation-from-an-instance)를 보면 알 수 있는데 변수를 검색할때 우선순위가 descriptor -> instance variable -> non-data descriptor -> class variable 순으로 탐색을 하기 때문이다. 

그래서 만약 변수명을 동일하게 구성하고 싶은 경우에는 이를 반드시 알고 있어야 한다.

기본 개요는 이 정도면 충분한 것 같아서 글은 여기서 마치도록 하겠다. 추가적인 세부내용을 알고싶다면 [https://docs.python.org/ko/3/howto/descriptor.html#](https://docs.python.org/ko/3/howto/descriptor.html#)을 참고하길 바란다.

