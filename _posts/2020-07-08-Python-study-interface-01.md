---
title: 전문가를 위한 Python (p.391~419)
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
---

2020.07.08 18:00~20:00

* 보호된 속성과 비공개 속성은 인터페이스에 속하지 않는다고 정의되어 있다 -> override 하지 않는다?
* interface : 시스템에서 어떤 역할을 할 수 있게 해주는 객체의 공개 매서드
* '프로토콜'이라는 명칭은 스몰토크에서 시작
* 프로토콜은 비공식적 인터페이스 -> 공식 인터페이스처럼 강제할 수 없다.

* getitem 특별 매서드만 구현하고 iter는 구현하지 않은 경우도 python 인터프리터가 iter를 대체하여 정수 index로 getitem을 호출하기 때문에 시퀀스로서 기능한다. 비슷하게 contains를 구현하지 않았더라도 python 인터프리터가 전체 항목을 조사하기 때문에 in 연산자도 사용할 수 있음. -> 파이썬은 최선을 다한다!!!

* 멍키 패칭 : 함수를 별도로 만들어서 런타임에 이어 붙임. (실제로 사용할 일은 없을 것 같음... 호출하는 그 순간에 검사한다는 파이썬의 특징만 다시 확인)

* 줄줄 이어진 if/elif/else 블록에 isinstance를 사용하는 것은 좋지 않다. 인터프리터가 적절한 매서드를 호출할 수 있도록 해야 한다.

```python
# field_names에는 str이 들어울 수도 있고 아닐 수도 있음.
try:
    field_names = field_names.replace(',','').split()
    # 그런데 일단 냅다 str이라고 가정하고 replace와 split을 호출함.
    # replace나 split을 호출할 수 없는 경우(str이 아닌 경우)라면
    # 에러가 나옴.
    # str인 경우에는 정상적으로 field_names 값이 갱신됨
except AttributeError:
    # field_names가 str이 아닌 경우에는
    # 애초부터 갱신될 필요가 없었음. pass
    pass
field_names = tuple(field_names)
# 이제 split까지 된 field_names는 원래 str이었든 아니었든 tuple로 만들 수 있음.
```

좀 ?는 남는다... exception을 발생 시키는게 isinstance() if 구분 하나 넣는 거보다 더 나을 정도로 프로그램에 부담이 없단 말인가...? exception은 자원 잡아먹는 거 아니었나...

* ABC를 상속할 때는 추상 메서드를 모두 구현해야 한다. 하나라도 구현되어 있지 않다면 객체를 생성하려 시도할 때 TypeError가 발생.
* 상속한 메서드를 더 효율이 좋은 메서드로 override 하는 것도 가능하다.

Collections.abc

* interable, contrainer, sized : iterable은 iter를 통해 반복을, container은 contains를 통해 in 연산자를, sized는 len을 통해 len()메서드를 지원한다.
* Sequence, mapping, set : 각각 가변형 서브클래스를 가짐. (ex. MutableSequence)
* MappingView : python3에서 items(), keys(), values() 메서드에서 반환된 객체는 각기 ItemsView, KeysView, ValuesView를 상속한다.
* Callable, Hashable : 호출할 수 있는지는 callable()함수를 사용할 수 있지만 Hashable은 이와 동등한 함수가 없음. isinstance(my_obj, Hashable)과 같이 사용.
* Iterator : Iterable을 상속.

numbers 패키지

* Number 숫자
* Complex 복소수
* Real 실수
* Rational 유리수
* Integral 정수(integral number = integer 인 듯?)

decimal.Decimal은 numbers.Real의 가상 서브 클래스가 아님에 주의.

### ABC의 정의와 사용

* 추상메서드는 def위에 데커레이터로 @abc.abstractmethod와 같이 표시.
* 추상 매서드도 실제 구현 코드를 가질 수 있다. 이 경우에는 ABC에 정의된 인터페이스만 사용해야 한다. 서브클래스에서는 super()를 이용해서 추상 매서드가 구현한 기능을 재사용할 수 있다.
* docsting만 쓰고 끝내는 경우도 많다.
* 특정 매서드가 특정 에러를 발생시킨다는 것도 interface의 일부이지만 이는 docsting에서만 선언할 수 있다.
