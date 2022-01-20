---
title: 전문가를 위한 Python (p.569~598)
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
---

2020.11.17  
2020.11.24  
2020.12.25  

#### Coroutine의 네가지 상태

* `GEN_CREATED` 시작하기 위해 대기
* `GEN_RUNNING` 현재 인터프리터가 실행되고 있는 상태
* `GEN_SUSPENDED` 현재 yield 문에서 대기하고 있는 상태
* `GEN_CLOSED` 실행이 완료된 상태

```python
>>> def simple_coroutine():
        print('started')
        x = yield
        print('received:', x)

>>> my_coro = simple_coroutine()  # 처음 생성. 이 시점에서 GEN_CREATED
>>> next(my_coro)   # 한번 next를 해서 yield를 만나게 해야한다. 안하면 send 호출 불가.
started             # started를 프린트하고 yield를 만나 나온 상태. GEN_SUSPENDED.
>>> my_coro.send(1) # send로 값을 보낸다.
received: 1         # 보낸 값이 yield 자리에 들어가서 x가 되고 received가 찍힘.
# 이 시점에 다음 yield 끝까지 못가고 generator가 종료되며 StopIteration Error 발생
```

#### @coroutine decorator

```python
from functools import wraps

def coroutine(func):  # averager가 이 func로 들어온다
    @wraps(func)
    def primer(*args, **kwargs):
        gen = func(*args, **kwargs)
        next(gen)
        return gen
    return primer  # primer라는 함수를 만들어 얘를 리턴.

@coroutine
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield average
        total += term
        count += 1
        average = total/count

>>> coro_avg = averager() 
>>> averager
<function averager at 0x7f42049ab550>
>>> coro_avg = averager()
>>> coro_avg
<generator object averager at 0x7f42032c3200>
>>> coro_avg.send(10)
10.0
>>> coro_avg.send(30)
20.0
>>> averager.__name__  # wraps라는 decorator를 별도로 써준건 이와 같은 결과를 의도한 것 이었다. 내가 원하는 값이 정확히 출력되도록. 그리고 primer라는 함수 명 자체도 자주 사용되는 것 같았음.
'averager'
```

#### coroutine에 명시적으로 예외를 전달하는 throw와 close

* [generator.throw(type, value=None, traceback=None)](https://www.python.org/dev/peps/pep-0342/#new-generator-method-throw-type-value-none-traceback-none)
* [generator.close()](https://www.python.org/dev/peps/pep-0342/#new-generator-method-close) : yield 표현 식이 GeneratorExit 예외를 발생시킨다. 코루틴이 종료되어 `GEN_CLOSE` 상태로 간다.

```python
class SampleException(Exception):
    pass

def exc_handling():
    print('coroutine start...')
    while True:
        try:
            x = yield
        except SampleException:  # .throw(DemoException)과 같은 식으로 호출하면 이쪽으로 들어온다.
            print('Exception handling here')
        else:
            print('coroutine received...', x)  # 그 외의 경우는 여기로 들어옴.
    raise RuntimeError('FATAL ERROR')  # except로 잡지 않은 error는 이쪽으로. 들어오지 말아야 함. 에러가 처리되길 원한다면 while문을 try~ finally 구문 안에 넣어서 finally 단에서 처리 되도록 해야 한다.
```

#### yield from

* 다른 언어의 await와 비슷.

```python
>>> def gen1():
...     for c in 'AB':
...         yield c
...     for i in range(1, 3):
...         yield i
...
>>> def gen2():
...     yield from 'AB'  # gen2가 yield from 'AB'구문을 실행하면 'AB'이 값을 생성해 gen2를 호출한 값에 반환. 실질적으로 'AB'가 직접 호출자를 이끄는 것. 그러는 동안 gen2는 'AB'가 종료될 때까지 실행을 중단함. - 책이 쓰여진 시점에서는 PEP 492에서 async과 await 구문을 사용한 코루틴이 제안된 상태임.
...     yield frome range(1, 3)
,,,
>>>
```

* yield from의 가치는 중첩된 제너레이터를 복잡하게 사용할 때에 나타남.(PEP 380 - 하위 제너레이터에 위임하기 위한 구문)
* yield from의 특징은 가장 바깥 쪽 호출자와 가장 안에 있는 하위 제너레이터 사이에 양방향 채널을 열어준다는 것.
* `대표 제너레이터 - delegating generator` yield from <반복형> 표현식을 담고 있는 제너레이터 함수
* `하위 제너레이터 - subgenerator` yield from 표현식 중 <반복형>에서 가져 오는 제너레이터.
* `호출자 - caller` 대표 제너레이터를 호출하는 코드

```python
Result = namedtuple('Result','count_average')

# 하위 제네레이터
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        if term is None: 
            break
        total += term
        count += 1
        average = total / count
    return Result(count, average)

# 대표 제네레이터
def grouper(results, key):
    while True:
        results[key] = yield from averager()

# 호출자
def main(def):
    results = {}
    for key, value in data.items():
        group = grouper(results, key)
        next(group)  # 한번은 next 해서 제네레이터 기동시킴
        for value in values:
            group.send(value)  # send를 해서 보낸 value는 grouper를 통과하여 yield from으로 이어전 averager의 yield로 감
            # 이 send되는 과정을 grouper는 보지 못함.
        group.send(None)  # 마지막으로 None을 보내서 종료시키면 averager가 종료되어 Result가 리턴되어 yield from 자리로 들어가서 results[key]에 할당됨
```
