---
title: JavaScript에서 startsWith이 제대로 동작하지 않을 때
categories:
  - tinytip
tags:
  - javascript
  - startsWith
  - encoding
  - NFD
  - NFC
---

![E0lxn-eUcAAW51B](https://user-images.githubusercontent.com/8686251/207360190-f82fa6f1-c443-48b6-8a3e-da63cb528bc0.JPG)

javascript로 admin 페이지를 만들면서 업로드되는 파일을 파일명으로 검사하려고 했다.
그런데 분명 파일명이 맞는데 계속 false를 리턴하는 것이다. 개발자모드로 들어가서 복붙으로 해봐도 false가 나왔다. 환장 할 노릇이었다. (입력하면 또 true)

charAt을 이용해서 for loop를 이용해서 한글자 한글자 직어보았더니 자소가 분리되어서 나왔다. 인코딩 문제라는 생각이 들기 시작했다.

javascript는 NFC, NFD, NFKC, NFKD의 표준 정규화 방식을 사용하는데 NFD는 모든 음절을 분해하여 저장하고 NFC는 결합하여 저장하는 방식이라고 한다. 그리고 파일명을 비교할 string은 코드에 하드코딩 되어 있었는데 이것과 파일명을 읽어왔을 때의 형식이 맞지 않는 것으로 보였다.

```javascript
fileName.normalize(); // 괄호 안에 형식 지정 가능
```

normalize를 한번 호출해서 고칠 수 있었다.
