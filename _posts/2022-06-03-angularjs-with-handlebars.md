---
title: handlebars value missing에러 (feat.Angularjs와 Handlebars를 동시에 사용해야 할 때)
categories:
  - tinytip
tags:
  - Angularjs
  - Handlebars
  - 치환자
---

오래된 사내 Admin에 새 페이지를 추가해야하는 일이 생겼다. 다른 페이지들을 참조해서 잘 만들었는데, label을 사용해서 변수 값을 노출하는 부분만 넣으면 화면이 뜨지 않는 문제가 있었다. bind해서 쓰는 것은 문제가 없었는데 `{{변수명}}` 이렇게 바로 쓸 때만 문제인 것 같았다.

실제로 로그중에 `handlebars value missing`라는 에러가 떳고, 나는 `{{}}`를 AngularJs의 Expression으로 사용하려고 한건데 이걸 handlebars로 우선적으로 인식하고 있다는 생각이 들었다.

이는 AngularJs와 handlebars의 치환자가 동일해서 생기는 문제로 둘 중에 하나의 치환자를 변경해주는 방법으로 해결하는 경우가 많았다. 나의 경우는 모듈에 config를 추가해서 `[[]]`를 치환자로 사용하도록 하고 파일에서 `{{변수명}}` 대신 `[[변수명]]`을 사용하도록 해서 해결했다.

```js
MyModule.config(function($interpolateProvider) {
    $interpolateProvider.startSymbol("[[").endSymbol("]]");
});
```
