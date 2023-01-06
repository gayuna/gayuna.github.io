---
title: 'Google Apps Script : Gmail에서 안읽은 메일 제목 스프레드 시트에 정리하기'
categories:
  - GAS
tags:
  - google apps script
  - 구글 앱스 스크립트
  - 앱스 스크립트
  - 스프레드시트
---

1:1 시간에 구글만으로 batch를 만들 수 있다고 해서 찾아보니 정말 재밌는 기능이 있었다.
자바스크립트는 써본 적 없지만 눈치로 조금 써보기로. 1단계로 Gmail에서 타이틀 읽어서 스프레드 시트에 정리하기. 연습으로는 Daily Skimm이라는 뉴스레터의 제목을 읽기로 했다.

```javascript
function myFunction() {
  // find unread messages as list of threads
  var threads = GmailApp.search('is:unread');

  // iterate threads
  threads.forEach (function(thread) {
    // get first mail of the thread
    var mailSubject = thread.getFirstMessageSubject();

    // if the title doesn't match my purpose, skip.
    if (!mailSubject.includes("Daily Skimm")) {
      return;
    }

    // handle title to get desirable value
    var todaysSubject = mailSubject.split(":")[1]
    console.log(todaysSubject)
    
    // get spread sheet object to save
    var ss = SpreadsheetApp.openById("시트아이디");
    var sheet = ss.getSheetByName("title")

    // I want to save the string in the very first row. make a room.
    sheet.insertRowBefore(1)

    // set value
    var cell = sheet.getRange(1,1);
    cell.setValue(todaysSubject)

    // mark the message as read
    thread.getMessages()[0].markRead();
  });
}
```

먼저 읽지 않은 메일을 가지고 온다. 이 때의 리턴 값은 메일 thread의 리스트이다. 뉴스레터는 스레드의 형태를 가지지 않으므로 내가 읽을 스레드는 length가 1인 스레드일 것이다. 리스트를 순회해가면서 첫번째 (그리고 유일한) 메세지의 타이틀을 가져온다. 내가 원하는 메일은 제목에 `Daily Skimm`이라는 글자가 들어가야 하므로 들어가있지 않다면 람다 함수를 종료해서 다음 항목으로 넘어간다.

Daily Skimm이 들어가는 경우는 내가 원하는 포맷으로 메일 제목을 변경한다. 여기서는 앞의 `Daily Skimm:`을 빼고 그날의 제목만 저장하려고 했다. `:`가 특징이므로 `:`로 split을 하고 두번째(index=1) 값을 사용한다.

저장할 스프레드 시트를 불러온다. 여기서는 간단히 Id를 가져와서 바로 열었다. 저장할 시트는 'title'이라는 이름으로 이미 만들어놔서 `getSheetByName`으로 가져와서 사용한다.

가장 최신의 메일이 가장 위에 오길 바라기 때문에 한줄을 맨 위에 새로 삽입하고 원하는 값을 넣는다.

같은 메일이 두번 읽히는 것을 방지하기 위해 읽음 처리를 하고 종료한다.

뭐 써야하는지 알겠으니까 굴러가게 만든 다음에는 한번에 읽어서 한번에 쓰고 종료해서 접근 횟수를 줄여서 속도를 올릴 수 있겠다.
