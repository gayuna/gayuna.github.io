---
title: bash에서 파라미터를 받는 alias 만들기
categories:
  - tinytip
tags:
  - bash
  - alias
---

일을 하다보면 수많은 파일이 쌓이는데 특히 자동으로 생성되는 로그파일들이 그렇다. 크래시의 경우는 더 그렇고. 한번에 지우고 싶은데 지워야 하는 것들과 지워서는 안되는 것들이 모여있는 경우는 지우기 까다로웠다. wildcard를 넣어서 recursive하게 find해서 지우되 그걸 매번 find에 옵션넣고 하는게 귀찮아서 그냥 parameter를 받는 alias를 만들면 되지 않을까 싶어서 구글링을 했다. 그래서 찾은게 [이 답변](https://stackoverflow.com/a/42466441/7701097)의 solution 1.

```bash
$ alias wrap_args='f(){ echo before "$@" after;  unset -f f; }; f'
$ wrap_args x y z
before x y z after
```

설명 된 원리는 간단하다. 순식간에 함수 f를 정의해서 사용하고 그걸 unset까지 하는거다. 저 $@ 부분에 내가 넣는 파라미터가 들어간다. find 함수화 결합해보니 잘 동작한다. 앞으로도 이걸 여러 커맨드에 자주 사용할 것 같다.
