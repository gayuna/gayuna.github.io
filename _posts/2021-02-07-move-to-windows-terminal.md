---
title: 30분만에 windows terminal로 옮긴 후기
excerpt_separator: "<!--more-->"
categories:
  - tinytip
tags:
  - terminal
---

2015년에 인텔 i5, RAM 4G 컴퓨터를 지급받고 경악했는데, 그 컴퓨터를 2021년 1월에서야 교체 받았다. 친구는 이건 교체라기보다는 컴퓨터가 나 이제 그냥 죽여줘... 하고 죽은 거라는데 작년 하반기에는 일주일에 1~2회의 블루스크린을 겪어서 딱히 반박할 의사도 의지도 없었다.

여튼! 어차피 새로 로컬 개발환경 설치하는 김에 조금씩 바꾸고 있다. 작년에 vscode 환경은 꾸준히 업데이트했지만 그 외에는 관성적으로 쓰고 있었기 때문인데, windows terminal은 처음 preview 버전 나왔다는 이야기만 듣고 좋은가? 싶었는데 실제로 써보니 너무 좋아서 30분 만에 이전을 완료했다.

이전 환경에서는 putty로 ssh 연결을 하고, WSL은 별도로 터미널을 열었었다. 이 putty를 관성적으로 쓰고 있긴했지만 항상 불만이었던 게 ssh key 방식이 putty만 다르다는 것. 그래서 변환 작업이 필요했을 뿐만 아니라 vscode등에서 쓰는 .ssh 아래 config를 재활용하지 못했다. 또한, pageant를 항상 실행시켜줘야 하므로 windows startup 폴더에 pageant가 인증서를 물고 들어가도록 설정을 해줬었다. (한번 해두면 안 해도 되긴 하지만 트레이의 아이콘이 거슬리고 어쨌든 불쾌한 환경임에는 틀림없다.)

windows terminal로 옮기려고 했던 이유는 WSL과 ssh remote를 탭만 바꿔가며 한군데 안에서 다 할 수 있다는 것. 그리고 ssh config파일을 재활용 가능하다는 점에서 향후 개발환경이 일관적으로 유지될 수 있겠다는 것.

#### ssh 연결용 프로필 설정

Windows terminal 실행 후 탭의 + 옆 아래 화살표 같은 버튼을 누르면 설정 메뉴가 있다. 이를 클릭하면 setting을 위한 JSON 파일이 실행된다.

나는 vscode의 remote development를 위한 ssh 설정이 있었고 이를 그대로 사용하길 원했다. 그래서 "profiles" 아래 "list" 밑에 새로운 항목을 추가해줬다.

```json
            {
                "guid": "{07b52e3e-de2c-5db4-bd2d-ba1405140531}",
                "hidden": false,
                "name": "remote connect",
                "source": "ssh.exe My_linux_machine"
            }
```

마지막 줄이 포인트인데 `My_linux_machine`이 내가 가지고 있던 config 이름이고 이를 ssh.exe의 인수로 주었다. 맨 위의 guid는 다른 거를 복사하고 난 후 끝에 숫자들을 바꿔줘서 고유한 값으로 만들었다.

마지막으로 setting.json 파일 맨 위쪽의 "default profile" 항목을 저 guid 값으로 변경하면 이제 windows terminal을 실행할 때 바로 My_linux_machine 설정대로 연결한다.

#### 내 취향대로 커스텀 하기

기본 테마는 이쁘지 않기 때문에 내 취향에 맞는 테마로 변경했다. 기존에 putty에서는 [Nord theme](https://www.nordtheme.com/)을 사용하고 있었는데 port 항목을 보니 windows terminal은 아직 공식적으로 지원하고 있지는 않았다. 하지만 검색해보니 [vscode 테마값을 조금만 수정하여 그대로 사용하는 방법](https://blog.anaisbetts.org/vs-code-themes-in-windows-terminal/)들이 많이 있었고, [Nord theme을 그렇게 수정한 값을 공유한 포스팅](https://compiledexperience.com/blog/posts/windows-terminal-nord)을 발견했다. 테마 변경 또한 settings.json에서 할 수 있다. 간단한 붙여넣기로 끝나는데, [자세한 방법을 설명해 놓은 블로그들이 많다](https://popcorn16.tistory.com/118).

그 외에는 자잘한 수정을 했다. "copyOnSelect"의 디폴트 값이 false인데 이를 true로 변경하고 폰트를 평소 사용하는 [D2Coding](https://github.com/naver/d2codingfont)으로 변경하고 모니터 해상도에 맞춰 사이즈를 조절했다.

#### 개인 setting과 충돌 날 때

유일한 문제점은 tmux key mapping 설정과의 충돌이었는데 나는 Alt + 방향키로 pane을 바꾸도록 설정했는데 이게 먹히지 않았다. 검색해보니 [windows terminal의 default setting과 충돌 나서 나는 문제](https://github.com/microsoft/terminal/issues/4763#issuecomment-593439579)로 해당 조합을 unbind 해주는 것으로 해결되었다.

30분이라고 했지만, 공식 nord theme이 없길래 이 기회에 nord 말고 다른 거 써볼까 해서 찾아보던 시간이고 실제로는 10분만에 완성할 수 있다. 1주일간 써보고 putty를 더는 쓰지 않아도 되겠다고 판단해 삭제하고 이 포스팅을 쓴다! 지금은 WSL과 ssh와 로컬 커맨드라인을 다 같은 터미널 안에서 쓰고 있고 너무 만족한다. 몇 년 사이에 windows 10으로 개발하기 너무 편해져서 좋다.
