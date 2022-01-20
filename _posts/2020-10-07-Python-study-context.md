---
title: 전문가를 위한 Python (p.551~563)
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
---

2020.10.07

#### Else block

* for 다음에 오는 else block은 for loop에서 break문으로 나오지 않은 경우 진입한다.  _검색할 때 `i<n` 해놓고 `i==n`이면 못찾음 이런 코드 대신 활용할 수 있겠군_
* while 다음에 오는 else block은 while loop에서 break문으로 나오죄 않은 경우 진입한다.
* try block에서 예외가 발생하지 않은 경우 else block으로 진입한다. 이 때, else 블록 안에서 발생한 예외는 else 블록 앞에 나오는 except 블록에서 처리되지 않음에 주의.
  * try block 안에는 예외가 발생할 수 있는 가능성이 있는 코드만 들어가야 함에 주의.

#### EAFP

올바른게 있다고 가정하고 가정이 잘못되었을 때 예외를 잡아 해결한다.  
python coding 스타일인 것.  
C와 너무 달라서 당황했었는데 그게 언어의 철학이라니.  

#### With block

* with 문을 위해 context 관리자 객체가 존재한다.
* context 관리자는 try/finally 패턴을 단순화시키며, `__enter__`와 `__exit__` 메서드가 필요. with block이 시작될 때 `__enter__`가 호출되고 끝날 때 `__exit__`이 호출된다.

```python
>>> with open('mirror.py') as fp:  # context 관리자의 `__enter__`의 리턴값이 as 다음에 오는 fp에 바인딩 된다. 여기서 fp는 read/write를 하기 위한 class의 객체일 것. (TextIOWrapper) 만약 return이 None이었다면 as는 필요 없다.
...     src = fp.read(60)
...  # 이 시점에서 context 관리자의 `__exit__`이 호출될 것.
```

```python
class LookingGlass:
    def __enter__(self):
        import sys
        self.orig_write = sys.stdout.write  # 나중에 복구하기 위해/reverse_write함수에서 사용하기 위해 기존 출력 함수를 저장
        sys.stdout.write = self.reverse_write  # 출력 함수를 새로 만든 reverse_write로 바꿈
        return 'JABBERWOCKY'  # string을 리턴

    def reverse_write(self, text):
        self.orig_write(text[::-1])  # 들어온 text를 전부 거꾸로 바꿔서 orig_write에 넣음. 이는 `__enter__`에서 sys.stdout.write로 바뀌었으므로 출력될 것.

    def __exit__(self, exc_type, exc_value, traceback):  # exit 함수로 정해진 Parameter. 정상 종료의 경우는 None, None, None일 것이고 문제가 발생했다면 각각 해당하는 값들이 들어가 있을 것.
        import sys
        sys.stdout.write = self.original_write  # 원복
        if exc_type is ZeroDivisionError:
            print('Please DO NOT divide by zero!')
            return True  # True가 반환되어야 error가 처리된 것. 만약 True가 아닌 값이 리턴된다면 with block의 에러가 그보다 상위의 코드로 올라간다.
```

* `exc_type` _(exception type)_ : 예외 클래스 ex. ZeroDivisionError
* `exc_value` _(exception value)_ : 예외 객체.
* `traceback` : traceback 객체.

#### @contextmanager 사용하기

```python
import contextlib

@contextlib.contextmanager  #decorator 해놓고
def looking_glass():
    import sys
    original_write = sys.stdout.write

    def reverse_write(text):
        original_write(text[::-1])

    sys.stdout.write = reverse_write

    msg = ''
    try:
        yield 'JABBERWOCKY'  # yield 해놓으면 with문으로 돌아간다. 여기까지가 `__enter__`가 하던 작업을 하는 셈
    except ZeroDivisionError:
        msg = 'Please DO NOT divide by zero!'
    finally:
        sys.stdout.write = original_write  # 여기가 `__exit__`가 하던 작업을 함.
        if msg:
            print(msg)  # contextmanager decorator에서는 기본적으로 모든 error가 처리되었다고 생각한다. 만약 상위로 error를 올리고 싶다면 여기서 새로 exception을 만들어서 올려야 한다.
```
