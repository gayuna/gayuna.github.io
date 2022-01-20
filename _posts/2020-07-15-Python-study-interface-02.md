---
title: 전문가를 위한 Python (p.419~437)
categories: python
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
---

2020.07.15 18:30~19:30

* 어떤 class가 ABC를 상속하지 않더라도 그 클래스의 가상 서브클래스로 등록할 수 있다. 클래스에 ABC의 인터페이스를 다 구현해놓았을 것이라는 약속이 전제. python는 검사하지 않고 이를 믿는다. 만약 구현이 안되어있다면 Runtime error 발생.

```python

@ABCclass.register  # python 3.3 이후의 버젼에서 등록하는 방법
Class Mysubclass(list):
    def __init__(self):
        pass
```

```python
ABCclass.register(Mysubclass)  # python <=3.3 에서 등록하는 방법
```

* `__len__`만 구현되어 있으면 abc.Sized의 isinstance나 issubclass가 True로 반환됨. 이는 Sized class가 `__subclasshook__`이라는 특별 매서드를 가지고 있기 때문.

from [_collections_abc.py](https://github.com/python/cpython/blob/3.9/Lib/_collections_abc.py)

```python
def _check_methods(C, *methods):
    mro = C.__mro__
    for method in methods:
        for B in mro:
            if method in B.__dict__:
                if B.__dict__[method] is None:
                    return NotImplemented
                break
        else:
            return NotImplemented
    return True

class Sized(metaclass=ABCMeta):

    __slots__ = ()

    @abstractmethod
    def __len__(self):
        return 0

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Sized:
            return _check_methods(C, "__len__")
        return NotImplemented
```

C.__mro__에 나열된 클래스 (C와 C의 super class) 중 `__dict__`속성에 `__len__`이라는 속성을 가진 클래스가 하나라도 있으면 True를 반환해서 C가 Sized의 가상 서브클래스임을 알려준다.  
그렇지 않으면 NotImplemented를 반환, 서브 클래스 검사를 진행한다.

근데 기본적으로 `__subclasshook__`을 작성할 일 까지는 X...

여기까지 덕타이핑과 구스타이핑의 내용을 잘 이해했는가?
