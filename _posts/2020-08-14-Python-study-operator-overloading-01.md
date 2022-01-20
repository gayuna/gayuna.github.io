---
title: 전문가를 위한 Python (p.465~490)
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
---

Vector(3, 4) == [3, 4]가 True여야 할까 False여야 할까.
의도에 따라 다름.
근데 의도에 맞게 구현할 수는 있어야겠지? :)

* 내장 자료형의 연산자는 오버로딩 불가
* 새로운 연산자의 생성 불가 (기존 연산자 오버로딩만 가능)
* is, and, or, not 연산자는 오버로딩 불가 (단, `&`, `|`, `~` 비트 연산자는 가능)

* __언제나 새로운 객체를 반환해야 한다.__ Self를 수정하지 않고 적절한 자료형의 객체를 새로 생성해서 반환할 것.

#### 단항연산자

|.|메소드|설명|
|------|---|---|
|-|`__neg__`|단항 산술 부정. x=2일 때 -x=-2|
|+|`__pos__`|단항 산술 덧셈. 일반적으로는 x와 +x가 동일 (다른 경우로 나온 예시는 소수점 많이 넘어가면 컴퓨터 특성상 달라지는 것과 Counter의 특징에 관련된 것이었는데... 누가 이렇게 써 라는 느낌..)|
|~|`__invert__`|정수 형의 비트 반전. (2의 보수가 됨)|

#### 중위연산자 + overloading

`__add__` : 길이가 다른 경우 짧은 쪽에 0 padding으로 해서 더하는 편이 낫다.

```python
def __add__(self, other):
    try:
        pairs = itertools.zip_longest(self, other, fillvalue=0.0)
        returns Vector(a+b for a, b in pairs)
    except  TypeError:
        return NotImplemented
        # 자료형의 비호환성 문제 때문에 결과를 반환할 수 없을 때는 NotImplemented를 반환해야 한다. (TypeError를 그대로 반환하면 안되는 것에 유의)
        # 그래야 python interpreter가 아래 flow chart에 따라 최선을 다해 다음 단계를 수행함
```

other 와의 관계로 method가 정의되기 때문에 교환 법칙이 성립하지 않는다. 따라서 역순 연산자(여기서는 `__radd__`)를 구현해줘야 한다.

#### a + b의 flow chart

1. a에 `__add__` method가 있는지 검사, 정의되어 있다면 `a.__add__(b)`를 호출. return이 NotImplemented가 아니라면 해당 return 값 반환
1. a에 `__add__` method가 정의되어 있지 않거나, 함수의 return값이 NotImplemented였다면 b에 `__radd__`가 정의되어 있는지 검사, 정의되어 있다면 `b.__radd__(a)`를 호출. return이 NotImplemented가 아니라면 해당 return값 반환
1. b에 `__radd__`가 정의되어 있지 않거나 함수의 return값이 NotImplemented가 아니라면 Type error 발생

여기서 NotImplemented는 NotImplementedError와 다름에 주의. NotImplemented는 중위연산자(예를들어 +)가 피연산자(예를들어 a, b)를 처리할 수 없을 때 python interpreter에 반환하는 특별한 싱글턴 값. NotImplementedError는 추상클래스를 만들 때 구상클래스가 이를 꼭 override 해야 함을 알려주기 위해 발생시키는 Error 임.

#### 잠시 샛길

zip이 짧은 쪽에 맞춘다면 zip_longest는 긴쪽에 맞춘다.
zip의 return은 zip이었음. pop처럼 사용하면 텅 비어짐. zip은 len도 구현 안되어있음. enumerate도 zip처럼 return이 enumerate obeject임. 역시 len 구현 X

```python
>>> a = range(1, 5)
>>> b = range(1, 5, 2)
>>> c = zip(a, b)
>>> c
<zip object at 0x7fbe3ed1f240>
>>> len(c)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: object of type 'zip' has no len()
>>> for x in c:
...     print(x)
...
(1, 1)
(2, 3)
>>>
```

#### 중위연산자 * overloading

2 * (2, 3) = (4, 6)과 같은 스칼라 곱을 만드려고 함.

```python
    def __mul__(self, scarlar):
        if isinstance(self, numbers.Real):
        # 구체적인 자료형으로 하드코딩 하는 대신 ABC(numbers.Real)로 검사. ABC로 했으므로 nunbers.Real의 서브클래스, 가상 서브클래스를 모두 포함하게 된다.
            return Vector(n * scarlar for n in self)
        else:
            return NotImplemented

    def _rmul__(self, scalar):
        return self * scarlar  # 위의 __mul__이 호출됨
```

#### 비교연산자

==, !=, >, <, >=, <= 연산자.  
앞 내용과 비슷하지만 추가적으로

* 정방향과 역순에 동일한 세트의 연산자가 호출 됨. >(`__gt__`)에서 Not Implemented가 발생한다면 앞뒤 순서를 바꿔 <(`__lt__`)가 호출된다.
* ==와 != 연산자의 경우 역순 메서드가 실패하면 TypeError를 발생시키는 대신 __객체의 ID를 비교한다__

```python
    def __eq__(self, other):
        return ((len(self) == len(other)) and  # 길이가 다르면 어차피 다르고 zip해서 앞에만 같다고 True가 반환되면 안되므로 길이를 먼저 보는 중
                all(a==b for a, b in zip(self, other)))
```

이렇게 하면 Vector(1, 2, 3) == (1, 2, 3)은 True가 된다.
만약 이걸 False로 만들고 싶다면 return 앞에 타입 검사를 추가해준다.

```python
    def __eq__(self, other):
        if isinstance(other, Vector):
            return ((len(self) == len(other)) and
                    all(a==b for a, b in zip(self, other)))
        else:
            return NotImplemented
```

object에서 상속한 `__ne__` method가 `__eq__`가 구현되어 있고 이 결과가 NotImplemented가 아니라면 `__eq__`의 반대 값을 return해 주므로 별도로 `__ne__`를 구현하지 않아도 동작한다.

#### 복합 할당 연산자

in-place 연산자가 구현되어있지 않은 경우, a += b를 a = a+b와 동일하게 평가. 따라서 `__add__` method가 구현되어 있으면 += 연산자가 작동하게 됨.
별도로 구현하기 위해서는 `__iadd__`등의 in place 연산자를 구현한다. 새로운 객체를 생성하지 않고 피연산자를 직접 변경한다.
왼쪽 연산자의 내용이 갱신되므로 왼쪽 객체의 타입을 따라갈 것임이 명백하다.

```python
    def __iadd__(self, other):
        if isinstance(other, Tombola):
            other_iterable = other.inspect()
        else:
            try:
                other_iterable = iter(other)  # other를 interable로 만든다
            except TypeError:
                self_cls = type(self).__name__
                msg = "right operand in += must be {!r} or an iterable"
                raise TypeError(msg.format(self_cls))  # 저 !r 자리에 type의 name이 들어갈 것.
            self.load(other_iterable)  # exception에 걸리지 않은 경우 load 함수에 넣어서 추가하고
            return self  # self를 리턴한다. 복합 할당 연산자의 특징!! self 반환!!
```
