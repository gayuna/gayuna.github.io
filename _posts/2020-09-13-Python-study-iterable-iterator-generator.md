---
title: 전문가를 위한 Python (p.499~539)
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
---

2020.09.03  
2020.09.09
2020.09.17

#### sequence

파이썬 인터프리터가 x객체를 반복할 때는 iter(x)를 호출.
iter(x)는 다음과 같은 순서를 따른다.

1. 객체가 `__iter__` 메서드를 구현하는지 확인하고 이 메서드를 호출, 반복자를 가져온다.
1. `__iter__`메서드가 구현되어 있지 않더라도 `__getitem__`이 구현되어 있다면, 파이썬은 인덱스 0에서 시작해 항목을 순서대로 가져오는 반복자를 생성한다.
1. 이 과정이 모두 실패하면 파이썬은 'TypeError: 'C' object is not iterable'이라는 메시지와 함께 TypeError가 발생한다. 여기서 C는 대상 객체의 클래스이다.

모든 파이썬 시퀀스는 반복할 수 있지만,
`__iter__`특별 메서드를 구한 한 경우만 iterable이라고 간주한다.

* 파이썬 3.4까지는 Iterable인지 확인할 수 있는 확실한 방법은 iter(x)를 호출하고 TypeError를 체크하는 것. isinstance(x, abc.Iterable)은 `__getitem__`을 확인하지 않기 때문.

#### Iterable과 Iterator

반복형 Iterable
iter() 내장 함수가 반복자를 가져올 수 있는 모든 객체와 반복자(Iterator)를 반환하는 `__iter__` 메서드를 구현하는 객체는 iterable이다. 0에서 시작하는 인덱스를 받는 `__getitem__` 메서드를 구현하는 객체인 sequence도 마찬가지다.

Iterable이 Iterator를 생성.
Iterator는 Iterable을 상속받으며 `__next__`와 `__iter__`를 가지고 있음. 이 `__iter__`는 self를 리턴 함.

* Iterator인지 확인할 수 있는 가장 좋은 방법은 isinstance(x, abc.Iterator)를 호출하는 것이다.

```python
>>> s3 = Sentence('Pig and Pepper')  # s3는 iterable
>>> it = iter(s3)  # iterable은 iter를 가지고 있음 iter의 리턴은 iterator므로 it은 iterator.
>>> it
<iterator object at 0x...>
>>> next(it)  # iterator는 next 매서드를 가지고 있음.
'Pig'  # 첫 next였으니 첫번째 단어를 반환
>>> next(it)
'and'
>>> next(it)
'Pepper'
>>> next(it) # 더이상 없는데 next를 호출하면 오류가 남.
Traceback (most recent call last):
    ...
StopIteration
>>> list(it)
[]  # 다 비워진 iterator로 list를 만들어서 비워져 있음을 확인 가능
>>> list(iter(s3))
['Pig', 'and', 'Pepper']  # 물론 s3 본체에는 영향 X
```

반복자 Iterator
다음 항목을 반환하거나, 다음 항목이 없을 때 StopIteration 예외를 발생시키는, 인수를 받지 않는 `__next__` 매서드를 구현하는 객체. 파이썬 Iterator는 `__iter__` 메서드도 구현하므로 Iterable이기도 하다.

#### 고전적인 Iterator

iterable collection과 iterator 객체의 관계를 볼 것.

```python
class Sentence:
    def __iter__(self):
        return SentenceIterator(self.words)  # Iterator 객체를 생성해서 반환

class SentenceIterator:
    def __init__(self, words):
        self.words = words
        self.index = 0
    def __next__(self):
        try:
            word = self.words[self.index] # index에 해당되는 단어를 가지고 온다
        except IndexError:
            raise StopIteration()  # 끝까지 다 갔으면 index error가 날테니 이를 stop iteration을 발생시키도록 한다.
        self.index += 1  # 다음번을 위해 index++
        return word
    def __iter__(self):
        return self  # Iterator의 iter 매서드는 self를 반환해야 함

```

Sentence class 안에 `__iter__` 외에 `__next__`도 구현해서 Sentence 객체를 Iterable이자 Iterator로 만들고 싶을 수도 있다. 그러나 그것은 잘못된 생각이다. Iterable은 자기 자신을 반복하는 Iterator로 작동하면 안 된다. 즉, `__iter__`를 구현하되, `__next__`는 구현하면 안된다. 한편 편의를 위해 반복자는 반복형이 되어야 한다. 이때 반복자의 `__iter__`는 self를 반환해야 한다.

#### Generator 함수

```python
    def __iter__(self):
        for word in self.words:  # self.words는 init에서 만들어둔 것
            yield word  # for loop를 돌면서 그때그때 다음 단어를 돌려보냄
        return
```

본체 안에 yield 키워드를 가지고 있으면 제네레이터 함수.
self.words를 객체 생성 시점에 만든다는 점에서 느긋한 구현은 아님.

```python
    def __iter__(self):
        for match in RE_WORD.finditer(self.text):  # finditer는 callable iterator object를 반환 -> 그걸 하나씩 따라감
            yield match.group()  # 매칭되는 텍스트 추출
```

__제네레이터 표현식은 지능형 리스트의 느긋한 버젼이다.__
지능형 리스트가 리스트 팩토리라면, 제네레티어 표현식은 제네레이터 팩토리다.

```python
>>> def gen_AB():
        print('start')
        yield 'A'
        print('continue')
        yield 'B'
        print('end.')

>>> res1 = [ x*3 for x in gen_AB()]  # 이렇게 하면 지능형 리스트를 만들면서 gen_AB를 호출해서 이 시점에 start, continue, end가 print 된다.
>>> for i in res1:
        print('-->', i)  # for 루프를 도는 시점에는 -->와 A, B만 각각 print 된다.
>>> res2 = ( x*3 for x in gen_AB() )  # ()에 묶인 애가 generator 표현식. 이 시점에서는 start / continue / end.가 뜨지 않았다.
>>> for i in res2:
        print('-->', i)
# 이 시점에서야 --> 알파벳과 start / continue / end가 번갈아가며 뜨게 됨.
```

따라서 `__iter__`를 다음과 같이 한줄로 쓸 수도 있다.

```python
    def __iter__(self):
        return (match.group() for match in RE_WORD.finditer(self.text))
```

여러줄에 걸친 코드가 필요하면 제네레이터 함수 사용. 위와 같이 한줄에 정리가 가능하다면 generator 표현식으로 충분할 것!

#### 등차수열 generator 만들기. 근데 이미 있을껄

```python
class ArithProgress:
    def __iter__(self):
        result = type(self.begin + self.step)(self.begin)  # type을 호출한 결과가 int가 나오면 int(self.begin) 요런식으로 될 것, return이 type이라는 것도 충격 그게 callable인 것도 충격
        forever = self.end is None  # None이면 True가 들어가서 무한 등차 수열
        while forever or result < self.end : # 무한대로 가거나 end보다 작을 동안
            yield result
            index += 1
            result = self.begin + self.step * index
```

재미있는 코드였지만 이미 파이썬이 itertools에서 제공해주고 있음.

```python
>>> import itertools
>>> gen = itertools.count(1, .5)  # 0.5가 아니라 .5로 써도 된다는게...
>>> next(gen)
1
>>> next(gen)
2
```

itertools를 활용하면 위의 함수를 더 간단하게 만들 수 있다.

#### 반복형을 reduce 하는 함수

* any와 all은 단락 평가 함수. __결과가 확정되는 즉시 반복자 소비를 중단__

```python
>>> all([])
True
>>> g = (n for n in [0, 0.0, 7, 8])
>>> any(g)  # 0, 0.0은 넘어가고 7을 만나서 True를 반환
True
>>> next(g)  # True 다음인 8을 반환
8
```

* sorted는 reversed와 달리 실제 리스드를 만들어서 반환한다. sorting을 한다는 것은 모든 항목을 읽어야 함 -> 모든 항목에 접근해야 함. 정렬한 후 새로 만들어서 반환.

#### iter() 함수 들여다보기

* 일반적으로 객체 x를 반복할 때 iter(x) 호출.
* 일반 함수나 콜러블 객체로부터 반복자를 생성하기 위해 두 개의 인수를 전달해서 호출 할 수도 있음.
* 첫 번째 인수는 인수 없이 반복적으로 호출되는 callable. 두 번째 인수는 sentinel로서 callable에서 이 값이 반환되면 반복자가 StopIteration 예외를 발생시킴.
* 이 방법을 사용해서 만든 객체는 소모하고 난 후에는 재사용될 수 없다.

```python
>>> def d6():
...     return randint(1, 6)
...
>>> d6_iter = iter(d6, 1)
>>> for roll in d6_iter:
...     print(roll)
...
4
3
6
3
>>> # 1이 나올 때까지 iter를 계속 돌면서 d6 함수를 호출하게 되어 있음. 여기에서 1이 나왔을 것임을 알 수 있다.
>>> d6_iter = iter(d6, 1)  # d6_iter를 또 쓰고싶으면 새로 생성해야 한다.
```
