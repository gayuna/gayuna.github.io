---
title: 전문가를 위한 Python (p.705~)
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
---

2021.01.31
2021.02.07

#### feed['Schedule']['evnets'][40]['name']보다 feed.Schedule.events[40].name

사용자 정의 dict 클래스를 구현해서 feed.Schedule.events[40].name 과 같은 식으로 접근한다. 재귀적으로 호출되어야 함에 유의.

`__getattr__()`메서드가 핵심이다. 이 메서드는 속성을 가져오기 위한 일반적인 과정이 실패할 대 인터프리터가 실행한다. (나에게 a가 없는데 my_instance.a 와 같은 식으로 접근하려고 할 때 호출된다는 뜻)

```python
class FrozenJSON:
    """
    처음에 name에 Schedule이 들어와서 FrozenJSON.build(Schedule)이 들어가서
    isinstance(obj, abc.Mapping)을 타서 그 return값에 다시 .을 해서 event를 하면
    FrozenJSON.build(events) 로 가서 isinstance(obj, abc.MutableSequence)로 가서
    리스트 안에 있는 애들에 대해 다 build(obj)를 했고 그 obj중에 하나가 name일 듯.
    """
    def __init__(self, mapping):
        self.__data = dict(mapping)  # 처음 객체를 생성할 때 일종의 매핑된 애를 받아서 그걸로 딕셔너리를 생성해서 데이터로 가지고 있는다.
    def __getattr__(self, name):  # .name 으로 접근했는데 name이 없어서 일로 온다
        if hasattr(self.__data, name):  # name이 가지고 있던 __data에 있는 속성이면
            return getattr(self.__data, name)  # __data에서 반환한다.
        else:
            return FrozenJSON.build(self.__data[name])  # name이 attribute에는 없지면 mapping dict의 key로는 존재 할 수가 있다. 있으면 호출을 할거고 없으면 key error를 낸다. 이를 위해 재귀적으로 FrozenJSON 객체를 만들어본다. -> 만들기 위한 build
    @classmethod
    def build(cls, obj):
        if isinstance(obj, abc.Mapping):  # __data[name]의 값이 Mapping이었다면 그대로 FrozenJSON을 return한다. 이 순간 feed.Schedule이 되고 그 다음에 .을 쓸 수 있게 되는 것.
            return cls(obj)
        elif isinstance(obj, abc.MutableSequence)
            return [cls.build(item) for item in obj]  # __data[name]의 값이 list였다면 리스트 안의 내용을 다 build를 재귀적으로 호출한 결과로 만든 list를 리턴한다. 이 순ㄱ나 feed.Schedule.events[40]과 같이 접근이 가능해진다.
        else:
            return obj  # JSON으로 왔으므로 둘 다 아니라면 그냥 obj return.
```

다만 위 예시에는 문제가 있는데 속성명이 파이썬 키워드 (예를 들면 `class`라든가 `int`라든가)라면 syntax error가 발생한다. 따라서 init 함수를 조정하는 것이 좋다. `keyword.iskeyword(key)`의 값이 true라면 뒤에 `_`를 붙인다든가.

#### `__new__()`를 이용하기

파이썬 class를 만들 때 흔히 `__init__`을 생성자라고 생각하지만 사실 생성은 `__new__`가 한다. (init은 말그대로 초기화를 하는 듯.) `__init__`은 class method이지만 특별 취급을 받아 @classmethod를 생략하며 반드시 객체를 반환한다. 이 반환된 객채가 `__init__`에 첫번재 인수 `self`가 된다.
위의 `build`메소드의 기능을 `__new__`안에 넣을 수 있다.

```python
class FrozenJSON:
    def __new__(cls, arg):
    """class 단위로 진행된다는 것만 빼면 위의 build와 내용 자체는 동일"""
        if isinstance(arg, abc.Mapping):
            return super().__new__(cls)  # 상위 클라스의 __new__를 호출해 객체를 생성한다.
        elif isinstance(arg, abc.MutableSequence):
            return [cls(item) for item in arg]
        else:
            return arg
    def __init__(self, mapping):
        self.__data={}
        for key, value in mapping.items():
            if iskeyword(key):  # python 키워드와 겹치는 키워드에 대한 예외처리. from keyworkd import iskeyword를 해줘야 한다.
                key += '_'
            self.__data[key] = value
    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data. name)
        else:
            return FrozenJSON(self.__data[name])  # 이제 그냥 생성자를 호출하면 재귀적으로 동착하게 된다.
```

이와 같이 구조화 했을 때의 문제는 검색 등이 쉽지 않다는 것. `feed.Schedule.events[40].speakrs`는 2471, 5199를 가지고 있지만 speakers는 이 번호대로 인덱싱 되지 않고 serial 속성의 값으로 정의되어 있기 때문에 하나씩 탐색해야 한다. venue 번호도 마찬가지.

이번에는 표준 모듈 pickle / shelves를 이용해서 위와 같은 문제를 해결해본다. [pickle](https://docs.python.org/ko/3/library/pickle.html)은 파이썬 객체 직렬화 포맷이고 [shelve](https://docs.python.org/ko/3/library/shelve.html) 안에 보관된다.

위에서 JSON 파일을 읽는 class를 만들었다면, 아래 예시는 JSON 파일을 읽어 shelve.Shelf에 저장한다. key는 event.33950과 같이 레코드 유형과 일련번호의 형태로, 값은 Record class의 객체로 한다.

```python
DB_NAME = 'data/schedule1_db'
CONFERENCE = 'conference.115'

class Record:
    def __init__(self, **kwargs):
        self.__dict__.update(kwargs)  # 객체의 속성들은 __dict__에 들어있는데 이 __dict__ 자체를 업데이트 하는 방법으로 우리가 저장하고 싶은 속성을 가지게 한다.
    
def load_db(db):
    raw_data = osconfeed.load()
    
    for collection, rec_list in raw_data['Schedule'].items():
        record_type = collection[:-1]  # record_type은 'event'와 같은 속성들
        for record in rec_list:
            key = '{}.{}'.format(record_type, record['serial'])  # event.33950과 같은 값을 만들어 key로 쓴다.
            record['serial'] = key  # 이 string으로 된 key로 serial field를 갱신하고
            db[key] = Record(**record)  # record로 Record 객체를 만들어 DB에 저장한다.
```
