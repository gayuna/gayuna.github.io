---
title: iTerm2에서 tmux prefix로 command+a 키 쓰기
categories:
  - tinytip
tags:
  - tmux
  - iTerm2
---

원래 맥북은 집에서 개인적인 용도로만 썼는데, 이번 회사에서는 개발 노트북으로 맥북 프로를 받아서 틈틈히 개발환경 셋팅중이다. 원래는 windows terminal을 썼었는데 일단 남들 다 쓴다는 iTerms2로 시작하기로 했다.  
알트 키가 사라지고 커맨드 키가 생긴건데, 이 커맨드 키를 못쓴다는게 너무 불편한거다. tmux는 키바인딩에 command키를 지원하지 않는다. 그렇지만 이 키를 안쓰기는 너무 아깝다.

#### .tmux.conf 설정

기존에 하던 대로 `.tmux.conf` 파일을 수정해주었다. ctrl + b를 unbind하고 ctrl + a를 새로 바인딩한다.

```
set-option -g prefix C-a
unbind-key C-b

bind-key C-a send-prefix
```

#### iTerms2 에서 command + a를 ctrl+a로 보내도록 설정

1. `command + ,`로 `Preferences`를 연다.
2. `Profiles`메뉴로 가서 우측 탭 중에 `Keys`를 선택하면 나타나는 서브메뉴에서 `Key Mappings`를 선택한다.
3. 여기서 `+`를 눌러서 Keyboard Shortcut에 `command + a`를 설정하고 (클릭한채로 버튼을 누르면 된다.) Action에 `Send Hex Codes`를 선택한 뒤 `1/0x1`을 입력한다.

#### 보낼 Hex Code를 알아내는 방법

이 Hex code가 인터넷에 있는 튜토리얼대로 해도 안되고 해서 좀 고생했다. 찾아낸 방법은 실제로 버튼이 어떻게 눌리는지 알아내는거였다. 앱스토어에서 `Key Codes`라는 어플리케이션을 깔아서 그 화면에서 Command + a를 눌러봤더니 Unicode란에 1/0x1이라고 나와서 그 값을 그대로 입력했다. 참고로 복사/붙여넣기 하면 안되었다. hex를 보낸다는 의미에선 hex값 그대로 뜨는 그게 더 맞을 것 같은데 왜일까... 같은 방법으로 커맨드+화살표 등도 새로운 명령으로 만들 수 있다.
