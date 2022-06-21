---
layout: single
title:  "[PYTHON] - self와 cls의 차이는? ( __init__ vs __new__ .. and __call__ )"
category: Python
tag: Python
---

Python에서 햇갈리는 부분 중 하나가 어떨때는 클래스 내부 인자가 cls이고,  
어떨때는 self로 사용이 된다는 점이다.

이 둘의 차이점은 무엇이고 왜 이렇게 표현을 하는 것일까?  
오늘은 이에 대해서 알아보자.

우선 self, cls의 차이를 논하기 전에 `__init__`과 `__new__`의 차이를 알아야 한다.

## `__init__`, `__new__` 의 차이점
파이썬에서 클래스를 정의하면 항상 `__init__`을 정의하고 거기에 인자를 초기화한다.  
그리고 이 상태로만 생성을 해두면 메모리 할당까지 자동으로 진행되기에 어떻게 메모리에 적재가 되는지에 대해서는 간과하게 된다.

결론부터 말하면 `__init__`은 이미 메모리 할당이 완료된 instance를 대상으로 멤버 변수들을 할당한다. 그래서 `__init__`에서는 메모리 할당과 관련된 부분은 일절없다고 봐야한다.

그렇다면 메모리 할당은 도대체 어디서 이뤄지는 것일까?

답은 바로 `__new__`에 있다. 인스턴스 생성시 파이썬에서는 `__init__`을 호출하기 전에 `__new__`를 항상 호출한다. 그리고 메모리 할당에 관한 동작은 모두 `__new__`에서 이뤄진다. 

일반적으로 우리가 만드는 class는 mutable type이기 때문에 `__new__`를 재정의할 필요없이 파이썬에서 자동으로 생성되는 `__new__`를 사용하면 된다. 허나 str, int, Unicode, tuple과 같은 __immutable type__은 `__new__`에 대해서 재정의를 해야하니 참고하자.

이를 응용하여 아래 코드와 같이 Singleton class를 만드는 것이 가능하다.

```python
class Singleton(object):
    def __new__(cls):
        if not hasattr(cls, 'instance'):
            cls.instance = super(Singleton, cls).__new__(cls)
        return cls.instance

s = Singleton()
s1 = Singleton()

assert id(s) == id(s1)
```

----

## 그렇다면 cls와 self의 차이점은?
cls와 self의 차이점은 class 함수인지, instance 함수인지에 따라 나뉘게 된다.  
cls의 경우에는 @class_method를 붙이거나, 혹은 `__new__` 함수 등에서 정의가 가능하고  
self는 일반적으로 함수를 했던것과 같이 정의를 하면된다.

`__new__`의 경우에는 @class_method를 붙이지 않고도 class 함수화 되어있는데, 이것은 compiler 내부에서 동작이 이뤄지는 듯 하다. 

>> Whenever a class is instantiated __new__ and __init__ methods are called. __new__ method will be called when an object is created and __init__ method will be called to initialize the object. In the base class object, the __new__ method is defined as a static method which requires to pass a parameter cls. cls represents the class that is needed to be instantiated, and the compiler automatically provides this parameter at the time of instantiation.


## __call__ method는 무엇일까?
`__init__`과 `__new__`와 같이 언급되는 `__call__`이라는 매직메소드도 존재한다.  
이것은 객체가 생성된 이후 `()` operator가 호출될때 불러오는 함수이다.  
아래 코드와 같이 `()` 호출시 `__call__` 함수가 호출 됨을 할 수 있다.  

```python
class A(object):
    def __call__(self):
        print('__call__')
    def __init__(self):
        print('__init__')
    def __new__(cls):
        print('__new__')
        return super(A, cls).__new__(cls)
        
a = A()
>>__new__
>>__init__

a()
>>__call__
```

관련해서 정확한 사용처는 잘 모르겠으나 metaclass를 사용한 Singleton 함수를 정의할때 아래와 같이 사용된다.

아래 코드에서 MyInt.__call__이 호출될 수 있는 이유는 int가 결국 MyInt()를 통해 생성된 클래스(객체)이고, int(4,5)가 호출될때 `()` operator를 호출하였기에 위와 같이 동작이 가능한 경우다.

```python
class MyInt(type):
    def __call__(cls, *args, **kwds):
        print('***** Here My int *****', args)
        print('Now do whatever you want with these objects...')
        return type.__call_(cls, *args, **kwds)

class int(metaclass=MyInt):
    def __init__(self, x, y):
        self.x = x
        self.y = y

i = int(4,5)
>> ***** Here My int ***** (4, 5)
>> Now do whatever you want with these objects...

```

이와 같은 metaclass의 특징을 이용하여 Singleton 패턴을 구현하는 것이 가능하다.

```python
class MetaSingleton(type):
    _instnaces = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(MetaSingleton, cls).__call__(*args, **kwargs)
        return cls._instance[cls]

class Logger(metaclass=MetaSingleton):
    pass

logger1 = Logger()
logger2 = Logger()
print(logger1, logger2)
```


## 참조글
- [geeksforgeeks](https://www.geeksforgeeks.org/__new__-in-python/) - __new__ in Python
- [pythontutorial.net](https://www.pythontutorial.net/python-oop/python-__new__/) - Python_new
- <<파이썬 디자인 패턴 2/e>> - 체탄 기리다 지음, 이우현 옮김