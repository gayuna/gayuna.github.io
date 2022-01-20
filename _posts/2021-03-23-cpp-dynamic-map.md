---
title: cpp의 map은 []연산자를 쓰면 기본 생성자를 호출한다.
excerpt_separator: "<!--more-->"
categories:
  - tinytip
tags:
  - cpp
---

본의 아니게 embedded C만 하던 사람이 C++를 쓰고 있는데, 이왕 쓸거면 C에 없던 자료구조를 써보자 싶어서 이것저것 건드려보고 있다.
와중에 2월에 했던건 동적으로 요소를 추가하는 Map+queue. 실행시에 들어오는 값들로 key값을 생성하고 Map에 존재하지 않으면 그 key로 새로운 큐를 할당받는다.
그리고 그 큐에 특정 자료형을 넣는 식이다.

검색해보니 map의 find 함수를 호출하고 리턴값이 발견하지 못한것으로 판단되었을 때 해당 key로 새로 insert하는 방식이 널리 이용되고 있는 것 같아서 그렇게 구현했다.
이게 2월의 이야기. 좀 더 깔끔하게 될 것 같은데?라는 생각은 했는데 일단 동작이 잘 되었으므로 다음으로 넘어갔다.

그런데 오늘 다른거 검색하다가 발견한 [이 답변](https://stackoverflow.com/a/42001375)을 보고 깜작 놀랐다. 
`just use the operator[] - it will create the value using its default constructor and return a reference to it. Or it will just find and return a reference to the already existing item without creating a new one.`
요악하자면 find나 get같은게 아니라 그냥 map[key] 로 접근하면 key에 대응되는 애가 있으면 걔를 리턴하고 없으면 기본 생성자로 만들어서 새로 리턴한다는 것.

어라라 싶어서 바로 코드에 적용했고 정말 잘 되어서 바로 업데이트 완료했다. 아니 이게 가능하면 애당초 왜 이런 방법이 더 널리 쓰이지 않는거지? 이렇게 오늘도 코드 몇줄 줄였다! 난 코드 줄이는게 제일 좋아잉.
