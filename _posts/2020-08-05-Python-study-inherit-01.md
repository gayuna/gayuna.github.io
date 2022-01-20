---
title: 전문가를 위한 Python (p.439~455)
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
---

2020.08.05 12:40~13:30

* Dict를 상속하는 경우 vs UserDict를 상속하는 경우

```python
class DoppelDict(dict):
    def __setitem__(self, key, value):
        super().__setitem__(key, [value] * 2)
```

이 경우

1. 생성 할 때 DoppelDict(one=1)과 같이 넣은 경우
1. ins['two']=2로 삽입한 경우
1. ins.update(three=3)으로 삽입한 경우

의 결과가 모두 다르다.

1. `__init__`은 `__setitem__`이 override 되었다는 것을 고려하지 않아서 1이 들어간다.
1. ins['two']는 `__setitem__` 매서드를 사용하므로 [2, 2]와 같은 형태로 들어간다.
1. update 메서드는 `__setitem__` 매서드를 사용하지 않으므로 3이 들어간다.

결과 : {'one':1, 'two':[2, 2], 'three':3}

instance를 기존 Dict를 이용해 복제하는 경우에도 override가 되지 않는다는 문제가 있다.

```python
class AnswerDict(dict):
    def __getitem__(self, key):
        return 42
```

와 같은 class를 만들었을 경우, 어떤 key-value의 값을 넣더라도 항상 42가 리턴될 것이라고 기대한다.
하지만

``` python
ad = AnswerDict(one=1)
d = {}
d.update(ad)
```

와 같이 새로운 dict를 만들어 사용자가 만든 dict 값으로 업데이트 해줬을 경우, `ad['one']`의 리턴값(42)과 `d['one']`의 리턴값(1)이 달라진다.

위와 같은 상황을 방지하기 위해서 collections.UserDict를 상속하면 일관성이 유지된다.

```python
class DoppelDict(collections.UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(key, [value] * 2)
```

```python
class AnswerDict(collections.UserDict):
    def __getitem__(self, key):
        return 42
```

이제 DoppelDict는 어떤 방식으로 넣어도 `[value, value]` 방식으로 저장되고 AnswerDict는 Dict로 복제해도 이 class로 만든 key:value의 리턴값은 항상 42로 고정된다.

* 메서드 결정 순서
이중 상속의 경우 어느 쪽으로 따라갈지는 MRO(Method Resolution Order, 메서드 결정 순서)를 따른다.
class의 `__mro__` 속성이 현재 클래스부터 object class까지의 슈퍼클래스들의 MRO를 튜플로 저장하고 있다.

```python
class A:
    def ping(self):
        print('ping')

class B(A):
    def pong(self):
        print('pong')

class C(A):
    def pong(self):
        print('PONG')

class D(B, C):  # B가 앞에 있으므로 B클래스에서 먼저 탐색하게 된다
    def ping(self):
        super().ping()  # A.ping(self) 와 같음.
        print('post-ping')

    def pingpong(self):
        self.ping()
        # D이 ping으로 가서 B로 가서 A의 ping에서 ping을 출력한 후 돌아와 post-ping을 출력
        super().ping()  # B로 가서 A의 ping에서 ping을 출력
        self.pong()  # B의 pong으로 가서 pong 출력
        super().pong()  # B의 pong으로 가서 pong 출력
        C.pong(self)
        # binding되지 않은 class에 직접 호출할 때는 self를 넣어준다.
        # C의 pong으로 가서 PONG 출력
```

memo:
mro 순서에서 최종 클라스와 가까운 순서대로 나오는게 아니라 한 super클래스의 superclass 이런식으로 타고 가기 시작해서 object까지 갔다가 다시 돌아와서 다음 상속받는 애 이런식으로 타고가는 듯 하다.

다중 상속 다루기

* 인터페이스 상속과 구현 상속을 구분한다.  
코드 재사용을 위한 상속은 구현에 관련. 구성이나 위임으로 대체 가능. 인터페이스 상속은 프레임워크에서 중추적인 역할을 수행 (더 공부 필요)
* ABC를 이용해서 인터페이스를 명확히 한다.  
interface인 경우는 꼭 ABC를 사용해서 interface임을 명시하라는 것.
* 코드를 재사용하기 위해 믹스인을 사용한다.  
`is-a`관계가 아니라 서로 관련 없는 여러 서브클래스에서 코드를 재사용하기 위해 설계된 클래스는 명시적으로 `Mixin` class로 만들어야 한다. Mixin class는 새로운 자료형을 정의하지 않는다. 재사용 할 매서드의 집합일 뿐 (Util같은?) Mixin class로 직접 객체를 생성하지 X. Mixin class를 상속하는 구상 클래스는 다른 클래스도 상속할 것. Mixin class는 밀접하게 연관된 매서드 몇개를 구현, 하나의 구체적인 행위를 제공해야 함.
* 이름을 통해 믹스인임을 명확히 한다.  
믹스인임을 표시할 다른 방법이 없으니 클래스명 맨 뒤에 Mixin을 붙일 것.
* ABC가 믹스인이 될 수는 있지만 믹스인이라고 해서 ABC인 것은 아니다.  
ABC는 구상 매서드(instance method)를 구현할 수 있으므로 믹스인으로 사용할 수 있다. __BUT__ ABC는 자료형을 정의하지만, Mixin은 자료형을 정의하지는 않는다. ABC는 다른 클래스의 유일한 base class가 될 수 있지만 Mixin만은 상속받지 않는다.
* 두 개 이상의 구상 클래스에서 상속받지 않는다.  
구상 클래스의 슈퍼 클래스 중 하나를 제외한 나머지 클래스는 ABC거나 Mixin이어야 한다.
* 사용자에게 집합 클래스를 제공한다.  
ABC와 Mixin을 조합해 코드에 유용한 기능을 제공할 수 있을 때는 이들을 통합한 집한 클래스(aggregate class)를 제공하는게 좋다.

```python
class Widget(BaseWidget, Pack, Place, Grid):
    pass  # Widget 자체에서 구현한 것은 없고 Pack Place 등을 묶음
```

* 클래스 상속보다 객체 구성을 사용하라.
