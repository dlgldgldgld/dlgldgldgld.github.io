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

**- Code**
```python

```

**- Result**
```text

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

