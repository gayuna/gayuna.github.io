---
title: Google technical writing One, 수업 후기
categories:
  - etc
tags:
  - Technical Writing
---

![E0lxnkBVoAMuY5f](https://user-images.githubusercontent.com/8686251/117090876-87338e00-ad94-11eb-825f-2bc0d16665e9.jpg)
앞서서 업데이트 했던 [구글 테크니컬 라이팅 페이지](https://developers.google.com/tech-writing)의 구글 수업에 참여했다. 미국 시간에 맞춘 강의들은 보통 한국시간으로 새벽이나 평일 낮인 경우가 많아 참여가 힘들었고, 이번에 뜬 강의는 유럽 시간에 맞춘 강의라 유럽시간 낮 1시 / 영국시간 12시라 한국시간 오후 8시여서 참가했다. 방법은 페이지 안에 앞으로 할 강의가 표로 정리되어 있는데, 해당 강의와 함께 있는 구글 Meet 링크를 클릭해 참여할 수 있다. 내가 참여한 수업은 영국 구글에서 일하는 테크니컬 라이터분이 진행하는 강의였다.

![1](https://user-images.githubusercontent.com/8686251/117091026-f7daaa80-ad94-11eb-852f-191a8eb4357d.png)
참여는 구글 드라이브로 이루어지기 때문에 구글 드라이브를 대충은 이용할 수 있는 편이 좋다. 사진과 같은 문서 링크가 제공되고, 해당 문서에서 즉석에서 Pair를 만든다. A, B, link의 세개 열이 있는데 특정 칸에 본인 이메일을 쓰면 같은 행에 메일을 쓴 사람들끼리 짝꿍이 된다. A인 사람이 제공되는 in-class 자료의 사본을 만들어서 함께 협업하면 되는데 나같은 경우는 짝꿍이 권한을 줄 줄 몰라서 내가 링크 공유해서 시작했다. 수업을 듣는 구글 미트와 별개로 짝꿍과 커뮤니케이션 할 수 있는 통로가 필요하다. 진행자가 추천하는 것은 별도의 구글 미트 채널을 뚫어서 링크를 공유하거나 구글 도큐먼트의 코멘트 기능을 이용하기 등이다.

![2](https://user-images.githubusercontent.com/8686251/117091200-7d5e5a80-ad95-11eb-858f-da7c5de6dc0d.png)
![3](https://user-images.githubusercontent.com/8686251/117091206-82bba500-ad95-11eb-9514-d7f8b223eea1.png)

수업은 강의를 짧게 듣고 문서의 연습 문제를 해보고 (2인이서 하기 때문에 문서 안에 연습 문제가 두벌씩 들어있다.) 서로의 답안을 보면서 아이디어를 공유해보고, 본인의 고칠 것이 있다면 다시 고친 후 진행자의 sample 답변을 보며 설명을 듣는 식으로 진행되었다. pre-class 자료를 충분히 읽었는데도 불구하고 바로 그 내용이 떠오르진 않아서 sample 답변을 보며 '아!!'하고 깨닫는 경우가 많았다. 예를 들면 `Performance optimization is overridden by the --noperf flag.`를 고칠 때 수동태를 능동태로 고쳐서 `The --noperf flag overrides the Performance Optimization.`라고 썼는데, 예시 답변은 `Specify the --noperf flag to turn off performance optimization.`같은 식이었다. 읽는 사람 입장에서 어떤 행동을 해야하는지 명령형으로 짧게 쓰고 `overrides` 같이 결과를 유추해야하는 단어보다는 `turn off`같은 직접적인 단어를 쓰는 것. 역시 연습과 실전은 다르구나 하는 생각이 들었다.

개인적으로 느낀 어려움은 어렵게 쓰여진 문장을 고쳐야 하는 경우, 복잡하게 쓰여진 문장 자체를 영어가 모국어가 아닌 나는 독해에 어려움을 겪었다... 문장을 이해해야 고칠텐데 애초에 문장의 의미가 파악이 힘든 경우. 그니까 테크니컬 라이팅 문서에서 말하는 '여러 국가의, 영어가 가장 편한 언어가 아닌 사람들이 읽을테니 간단한 영어를 사용하라'의 타겟이 나였던거지.

그리고 내 짝꿍은 수업의 절반이 좀 지났을 때 나가가지고 끝까지 같이 할 수가 없었다. 시간이 된다면 한번 One을 더 듣고 Two를 들어야겠다라고 생각했다. 일단 현재 나와있는 One의 스케쥴은 새벽 2시밖에 없어서 여름에나 들을 수 있지 않을까 싶고... 그때는 구글 Meet 사용해서 도전해봐야겠다.

![E0lxn-eUcAAW51B](https://user-images.githubusercontent.com/8686251/117091588-c19e2a80-ad96-11eb-871f-3d91e1f4d7ca.jpg)

이 슬라이드는 제법 재밌었다.

#### 번외: 새로 배운 단어

* `Minimum viable document set` 문서를 이렇게 가져가라고 했는데, 요지는 '사람들이 이해할 수 있을 정도로 충분하게, 하지만 유지보수 비용을 고려해 너무 길지는 않게 하라는 것. 문서에도 유지보수 비용이 든다는 것을 망각하고 무조건 자세하게만 적으면 안된다고. Minimum viable이라는 단어는 보통 product나 prototype에 자주 붙는 단어인 것 같다.
* `dogfooding` 혹은 `Eating your own dog food` 조직이 자신의 제품을 직접 써보는 것. 당연히 직접 써봐야 문제점도 발견할 수 있기 때문에. 스타벅스 알바 시절 음료수 하나씩 먹도록 해주던게 생각났다. 그 때도 그건 복지이기 이전에 먹어봐야 고객에게 추천도 가능해서라고 생각했기 때문에.
