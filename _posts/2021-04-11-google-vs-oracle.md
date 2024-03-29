---
title: Google vs Oracle 소송 판결문 읽기
excerpt_separator: "<!--more-->"
categories:
  - Translation
tags:
  - 오픈소스
  - 구글
  - 오라클
  - 공정사용
  - google
  - oracle
  - fairuse
  - java
  - API
---

`Disclaimer : 저는 법도 Java도 전문가가 아닙니다. 아래 내용을 사실과 다를 수 있으며 저는 그에 대한 책임을 지지 않습니다. 정확한 내용은 원본 문서를 참고해주세요.`

#### 배경

2005년, Google은 안드로이드의 JAVA에 SUN의 라이센스를 쓰기 위해 협상을 시작함. [SUN은 안드로이드가 다른 JAVA 프로그램들도 연결해서 쓸 수 있게 해달라고했으나 구글이 받아들이지 못하여 결렬되고 Google은 "clean room implementation"인 Dalvik을 개발하게 된다.](https://arstechnica.com/tech-policy/2012/04/sun-wanted-up-to-50-million-from-google-for-java-license-schmidt-says/) 이때 결렬의 주된 사유는 금전적인 부분이 아닌, Android에 대한 통제권에 대한 부분이었다. 구글은 안드로이드를 오픈소스화하기를 원했고 그 때문에 SUN에 통제권(의 일부)을 넘겨줄 수가 없었다. 2007년, 구글은 안드로이드를 발표하고 2010년, SUN을 인수한 Oracle이 JAVA의 저작권을 침해했다고 구글을 상대로 소송을 제기하게 된다.
다만 오픈 소스가 되길 원해서 SUN에게 통제권을 넘겨줄 수 없어서 결렬되었다는 주장은 반만 믿어야 할 것 같다. 구글이 본인들의 이익을 위해 Fully-control 하기 위해서 그런 것이라고 보는 것이 합당할 것. [그래서 Android도 GPL이 아닌 Apache 라이센스를 사용하고 있다.](http://www.fosspatents.com/2013/11/q-on-dec-4-oracle-v-google-android-java.html) 실제로 구글도 사건이 진행되는 동안 오픈소스와 관련된 논리를 펴지 않았으며, 이후 판결 중에도 공정 이용 여부를 판단할 때 오픈 소스로 공개한다는 것이 비상업적이라는 의미는 아니라는 판결이 나온다.

[저작권 침해가 되었다고 제기된 것은 37개 JAVA API의 선언부, range check 함수의 9줄, 8개의 테스트 파일이다.](https://guides.lib.umich.edu/c.php?g=791114&p=5747565)

#### [지방 법원 판결](https://www.leagle.com/decision/infdco20120601k39)

오라클의 주장은 `구조, 순서 및 구성(structure, sequence and organization)`을 따라 했다는 것. 1. API에 저작권이 존재하는가 2. 존재한다면 침해했는가 3. 침해했다면 공정 이용 사유에 해당하는가가 중점적으로 다뤄져야 하고 저작권 여부는 판사가, 나머지는 배심원이 판단하기로 한다. 배심원은 저작권이 존재한다는 가정하에서 저작권이 침해되었다고 보았으며 공정 이용에 해당하는가에 관해서는 결정하지 못했다. 한편, 판사는 API에는 저작권이 존재하지 않는다고 판결했다.
코드 내에서 선언과 호출을 해야 하는데, 이는 Java를 사용할 때에는 같은 방식으로 사용해야 한다. 예를 들어 `int a = java.lang.Math.max (2, 3);` 라는 코드는 java.lang의 Math 패키지에서 max라는 메소드를 호출하는 것이다. 특정 기능을 선언하기 위해서 특정 형식을 취해야만 하는 것이라고 볼 수 있다.
저작권법은 메소드를 구현하는 특정한 구현법에 대해서만 소유권을 인정한다. 다른 사람들은 자유롭게 같은 동작을 하는 다른 구현법을 만들 수 있다. 위의 max 예시에서도 최댓값을 구하는 다른 방법으로 구현을 찾으면 된다.  `메소드의 스펙은 아이디어고 구현이 표현이며 그 누구도 아이디어까지 독점할 수는 없다. (The method specification is the idea. The method implementation is the expression. No one may monopolize the idea.)` 또한 `일정한 기능을 수행하려면 일정한 형식을 갖추어야 하는데, 이런 경우는 저작권을 주장할 수 없고, 변수명이나 함수명도 이름과 짧은 문구는 저작권을 가질 수 없다는 법 때문에 저작권이 있다고 볼 수 없다. (merger and names doctrines)`
range check와 테스트 파일에 대해서는 0달러의 금액으로 판결 내렸다.

#### [항소 법원 판결](https://www.leagle.com/decision/infco20140509135)

항소 법원은 구조, 순서 및 구성이 저작권의 대상이 된다고 본다고 지방 법원의 판결을 뒤집었다. 공정 이용에 대해서는 다시 판단이 필요해 지방 법원으로 환송했다. merger doctrine에 관련해서는, 선언할 방법이 단 한 가지인 게 아니라면 선언부를 저작권법의 예외로 둘 이유가 없다고 판단한다. 구글이 복사한 선언문에 대해 선택/배열에 있어서 무제한의 가짓수가 존재했다는 것. 다른 안드로이드 코드들이 그렇듯, 충분히 다른 이름으로 정의될 수 있었다는 것.
또한 오라클이 단순 아이디어나 기능을 구성하는 패키지/클래스의 사용과 같은 것에 대해 저작권을 주장하는 것이 아니라, 37개의 Java API 패키지 각각의 이름을 지정하고 구성하는 특정 방식에 대한 저작권만을 주장하고 있다는 점에서 이런 시스템은 저작권의 대상이 될 수 있다고 판결 내렸다. 즉, 지방법원에서는 Java의 구조가 패키지-클래스-메소드로 구성되어 있으며 이는 다르게 사용할 수 없으므로 저작권이 없다고 판결하였지만, `Oracle이 주장하는 것은 단순히 패키지-클래스-메소드의 구조가 아니라 Oracle이 가지고 있는 '특정 패키지'의 '정확한 이름과 구조를 복사했다'`는 것.
구글은 상고했으나 대법원이 기각함.
이 시점에서 API에 저작권이 있다고 보고 저작권 침해에 대해 공정 이용이라고 볼 수 있는지로 소송의 중심이 옮겨간다.

#### 지방법원

저 사이트를 뒤졌는데 지방 법원 판결문을 찾지 못했다. JMOL을 거부한 이유는 봤으나 사건에 대한 판결문은 찾아내지 못했음. 다른 기사나 2심 판결문의 내용을 봤을 때 10명의 배심원이 만장일치로 공정 이용에 부합한다고 판결했고, 오라클은 배심원이 아닌 판사가 판결해야 한다고 주장했으나 이 또한 거부되었다.

[공정 이용은 다음의 네 가지 요건을 따지게 된다.](https://support.google.com/legal/answer/4558992?hl=ko)

1. 이용의 목적 및 특성 - 상업적인지, 교육적인지 등. 법정에서는 보통 변형적(transformative)인지를 따진다. 즉, 그대로 가져온 것이라면 변형적이지 않다고 봐서 공정 이용이라고 인정하기 어렵다.
1. 저작권 보호를 받는 저작물의 특징 - 사실을 서술한 저작물은 창작물보다 공정 이용이라고 인정받을 확률이 높다. 여기에서는 창의적(creative)인지를 주로 따진다.
1. 저작권 보호를 받는 저작물의 양, 비중, 가치 - 저작권 침해를 받는 부분이 전체 대비 비중이 크거나 양이 절대적으로 많다면 공정 이용이라고 판결을 받기 힘들다. 한편, 양이나 비중이 작더라도 그 부분이 전체에서 차지하는 가치가 크다면 이 또한 문제가 될 수 있다.
1. 저작권 보호를 받는 저작물의 사용이 시장이나 해당 저작물의 가치에 끼치는 영향 - 저작권을 침해해서 만들어진 것이 원저작물의 대체재가 되어 피해를 주었다면 공정 이용으로 인정받기 힘들다.

이 조건들에 대해 배심원들은

1. 구글이 상업적으로 사용하긴 했지만, 일부 요소만 스마트폰을 위해 사용했으며 구현부 코드는 직접 고유하게 구현했다는 데서 변형적이고
1. API의 선언부가 저작권의 보호를 받을 만큼은 창의적이지만 '엄청나게' 창의적이진 않으며 디자인에서도 기능적인 고려가 최우선시 되었고
1. 필요한 최소한의 정도로만 사용되었고
1. 기존 API는 desktop, laptop등의 시장이었던 것에 비해 smartphone 시장에서 쓰이므로 시장에서 해를 끼치지 않았다고 판단했다.

#### [항소 법원 판결](https://www.leagle.com/decision/infco20180327178)

오라클의 항소에 대해 항소 법원은 기존 판결을 뒤집으며 공정 이용 요건에 해당되지 않는다고 판결했다. 여전히 위의 4가지 요소의 판단이 핵심이다.

1. 이용의 목적 및 특성
    * 상업적 이용 - 구글은 안드로이드가 오픈 소스기 대문에 비상업적이라고 주장하고 오라클은 안드로이드는 높은 수익을 내고 있으며 이 수익성은 Java의 이용으로부터 온다고 주장했다. 재판부의 판단은 안드로이드가 무료라는 사실이 Google이 Java API 패키지를 비영리적으로 사용한다는 것은 아니며 구글이 수익은 안드로이드 자체가 아닌 광고에서 발생한다고 하지만 상업성은 수익을 얻는 방법에 의존하지 않는다. 즉 광고로 얻은 수익도 안드로이드로 얻은 것이라 볼 수 있다는 것. 심지어 공정 이용의 판단에 있어서 경제적 이익이 실제로 발생했느냐는 상업적임을 입증하는데 필수적인 조건이 아님.
    * 법에 정의된건 아니지만 기존 대법원 판결에서 ‘다른 목적이나 다른 성격을 가진 것을 새로 추가하고 새로운 표현, 의미 또는 메세지로 변경’하는 경우는 변형적이라 보고 공정 이용으로 보고 있음. 원본을 변경하거나 원본 작업과 다른 새로운 목적을 제공해야 함. 단순히 전체 중 일부만을 가져왔다고 해서 변형한 것이라 볼 수 없음. 안드로이드와 자바에서 해당 API들이 수행하는 역할은 동일함. 표현 내용이나 메세지를 변경한 것이 아니고 표현되는 형식만 변경된 것(데스크탑/서버에서 스마트폰으로)은 충분하지 않음. 또한 Java API는 안드로이드 이전에도 스마트폰에 존재했기 때문에 혁신적이라고도 볼 수 없음.
    * Bad faith - 구글이 업계 관행적으로 선언 코드와 SSO는사용할 수 있다는 선의의 믿음을 가지고 있었던 것임. 피고가 나쁜 의도가 없었다고 해서 책임이 없어지는 것이 아님. 악의로 하지 않았더라도 상업적이었다는 점, 변형이 없었다는 점에서 공정사용의 원칙에 해당하지 않음.
1. 저작물의 특징
    선언부와 SSO는 공정 이용이 아니라고 판단하기에는 창의성이 부족하다고 할 수 있음. 따라서 저작물의 성격은 공정 이용이라고 판단 됨. 하지만 두번째 요소는 전체 판단을 내릴 때 크게 중요한 요소는 아님.
1. 사용된 양과 비중
    복사된 자료가 표절자와 원본 작성자 모두에게 질적 가치가 있음. 지방 법원은 선언과 SSO만 복사하고 구현된 부분은 가져오지 않았기 때문에 최소한만 가져왔다고 판단했음. 그러나 구글이 복사한 11500라인 중 자바를 사용함으로서 작성 해야만 했던 부분은 170라인에 불과. 또한 구글은 전체 안드로이드 코드의 1%도 안된다고 주장하지만 37개 패키지에 대해 전체 SSO를 카피한 것이다. 구글은 시스템간의 일관성 유지를 위한 방법이었다고 말하지만, SW 개발자에게 익숙하게 만들어서 인기를 으려고 한 것이 공정 이용의 원칙의 이유가 될 수는 없음. 일부만 복사했다는 사실을 인정하더라도 그 내용이 안드로이드에서 중요했음. 다르게 API를 만들 수도 있었지만 결국 비지니스 관행상 비슷하게 가져간 것. 결론적으로 사용된 양과 비중을 판단하는데 있어서는 최대한 이해해줘도 중립정도라고만 판단할 수 있다.
1. 시장에서의 영향
    지방법원은 안드로이드의 선언부가 컴퓨터 시장에서 Java에게 해를 기치지 않았다고 판단함. 그 이유로 오픈 JDK라는 이름으로 모든 자바 API를 무료로 사용할 수 있게 했기 때문이라는 것을 들었음. 하지만 1. Java SE는 안드로이드 출시 이전에 이미 모바일 장치에서 사용되고 있었음. 안드로이드가 모바일 기기 시장에서 Java SE와 직접적으로 경쟁해서 시장에서의 지위를 약화시킴. 태블릿 시장에서도 아마존 킨들은 처음에 직접 자바를 라이센싱해서 사용했으나 안드로이드가 출시 된 이후에는 안드로이드로 옮겨감. 실제로 피혜를 입은 사례. 2. 또한 실제 피해가 없었다고 하더라도 공정을 사용을 판단할 때는 잠재적 피해에 초점을 맞추게 되어있음. 스마트폰은 개발 될 가능성이 높은 시장이었음. 구글은 Java가 HW를 만들 수 없었다고 하지만 이 시장은 다른 사람에게 라이센스를 판매하는 것으로 충분하기 때문에 굳이 기기를 만들 필요는 없음. 결국 네번째 요소는 공정 이용 원칙에 위배.

1, 4번 요소는 위배, 2번은 위배하지 않음, 3번은 중립이라면 전체적으로는 공정 이용의 원칙에 위배된다고 판결.

#### [대법원 판결](https://www.leagle.com/decision/insco20210405e91)

구글은 API가 저작권의 대상이 되는지, 또한 해당 케이스에서 공정 이용이라고 볼 수 있는지를 모두 포함하여 다시 상고했다. 대법원은 6-2로 구글의 Java API 사용이 (API에 저작권이 있다는 가정 하에) 공정 이용 범위 안에 있다고 판결했고 API가 저작권의 대상이 되는지는 판결하지 않았다. API의 저작권 적용 여부를 판단하지 않고 가정만 한 이유는 분쟁을 해결하는데 필요한 것 이상의 답변을 해서는 안되기 때문. 논증을 위해 저작권이 있다고 가정하고, 공정 이용 범위 안에 있다면 이미 저작권법의 처벌 대상이 아니기 때문에 저작권법이 적용 가능한가 여부는 판단 할 필요가 없다.

1. 이용의 목적 및 특성
    구글은 Java API의 일부를 정확하게 복사했으며 그 목적 또한 프로그래머가 특정 작업을 구현할 수 있게 했다는 것에서 동일하다. 하지만 여기서 멈춘다면 모든 저작권이 있는 컴퓨터 프로그램의 사용을 극단적으로 제한하게 된다. 따라서 컴퓨터 프로그램이 변형적인지를 판단할 때는 이용의 목적과 특성을 좀 더 구체적으로 파악해야 한다. 구글은 안드로이드와 관련된 API들을 재구현함으로 스마트폰 프로그램들이 유용하게 쓸 수 있게 하기 위해서만 Java API를 복사해왔다. 재구현함으로써 전혀 다른 컴퓨팅 환경에서 사용할 수 있는 새로운 도구들을 제공한 것이다. 역사적으로 인터페이스를 재구현 하는 것이 컴퓨터 프로그램의 개발을 더욱 발전시켰다. 그렇지 않다면 API 레이블이 변경 될 때마다 프로그램을 다시 짜야하고 프로그래머도 새로운 언어를 배워야 한다. 이 때문에 API의 재사용은 업계의 관행이며 Sun이 Java를 최초로 개발 할 때도 기존 인터페이스에서 가져온 것들이 많다. 이러한 배경에서 대법원은 구글이 Java API를 사용한 목적과 특성이 충분히 변형적이라고 판단한다.
1. 저작물의 특징
    선언부 코드는 다른 부분과는 다른 종류의 창의성을 가진다. Sun의 Java 개발자들은 코드를 선언할 때 쉽게 기억할 수 있도록 하려고 했다고 증언했다. Sun의 사업 전략도 API가 프로그래머들이 사용하는데 매력적으로 보이도록 하는데에 있다. 여러 증언에서도 유저 중심의 선언부와 혁신적인 작업이 들어가는 구현부 사이에 선을 긋는 모습들을 볼 수 있었다. 이런 점에서 선언부는 다른 컴퓨터 프로그램과 어느정도 다르다는 것을 볼 수 있다. 선언부의 가치는 저작권 자체에서 오는 것이 아니라 그것을 공부하기 위해 시간과 노력을 투자하는데서 온다. 저작권은 여러 종류의 글들 보호하지만 글에 따라 더 저작권의 중심에 가까운 것들이 있다(some works are closer to the core of copyright than others). 선언부는 저작권의 핵심과 상대적으로 멀리 있고 공정 이용의 원칙을 적용하는 것이 합당하다.
1. 사용된 양과 비중
    복사된 양은 Sun의 Java API 전체에 비하면 적은 양이다 (11500줄 / 286만줄). 여기서 문제는 이 11500줄을 전체 코드에서 볼 것인지 아니면 분리해서 봐야 할 것인지이다. 구글이 복사한 방법을 살펴봤을 때 이는 복사하지 않은 전체 코드를 고려하고 있음을 알 수 있다. 구글은 이 부분을 가져와서 안드로이드 스마트폰 플랫폼을 만드는데 사용했다. 다르게 구현했다면 이 프로그래머들을 데려와 플랫폼을 만들기 어려웠을 것. 항소심에서는 170줄만 카피함으로서 자바 호환성 목표를 달성할 수 있었을 것이라고 했지만, 이는 구글의 목표를 너무 좁게 간주한 것이다. 구글의 목표는 Java 언어를 그대로 사용하는 것에서 멈추는 것이 아니라 새 플랫폼에서도 그간의 지식과 경험을 활용할 수 있도록 하는 것이었다. 이처럼 복사된 부분이 한정되어있고, 변형적이며, 특정 목적을 위해 존재하는 경우는 '상당성(substantiality factor)'에서 공정 이용이 유효하게 작용한다. (The "substantiality" factor will generally weigh in favor of fair use where, as here, the amount of copying was tethered to a valid, and transformative, purpose.)
1. 시장에서의 영향
    구글이 API의 일부를 복사했든 하지않았든 Sun은 스마트폰 시장에 성공적으로 진입 할 수 없었을 것. 휴대폰 시장에 진출하기 위한 Sun의 노력은 성공적이지 못했고, Sun의 이전 CEO는 증언에서 스마트폰 시장 진출 실패의 원인이 안드로이드냐는 질문에서 아니라고 답했다. (Sun's former CEO was asked directly whether Sun's failure to build a smartphone was attributable to Google's development of Android, he answered that it was not.) 또한 구글의 안드로이드 플랫폼을 사용하는 모바일 장치와 Sun의 Java를 라이센싱한 장치가 다르다는 증언이 계속되었다. 많은 증언자들이 스마트폰 산업과 단순한 피쳐폰 산업을 구분 할 필요가 있다고 증언했다. 스마트폰 이전의 핸드폰은 터치스크린이나 쿼티키보드 같은 것을 가지지 않았다. 타블렛과 같은 기기에 있어서도 단순한 킨들은 Java 소프트웨어를 사용했으나 Kindle Fire는 안드로이드를 사용했는데, 이는 안드로이드가 더 고급 시장을 가지고 간다는 것을 보여준다. 즉, 안드로이드는 Java를 대체한 것이 아닌 전혀 다른 시장이었다.또한, Sun은 안드로이드에서 Java 언어에 훈련된 프로그래머가 이득을 가져다 줄 것이라고 봤는데, 이는 안드로이드와 Java를 다른 시장으로 보고 있었음을 알 수 있다.
    안드로이드의 시장성은 Sun의 Java가 아니라 안드로이드에 투자하는 쪽에 있다. Sun이 Java에 투자한 것과는 무관하다.
    또한 오라클의 저작권을 여기에 적용시키는 것은 대중에게 해를 끼칠 위험이 있다. 프로그래머에게 대체 API를 생성하는 비용과 어려움을 고려했을 때, 여기에 저작권을 적용하는 것은 미래의 창조성을 제한하고 이런 종류의 코드를 오라클만이 독점하게 할 수 있는 우려가 있다. 저작권의 범위를 여기까지로 제한 하는 것이 오라클의 저작권에 대한 권리를 해치지 않으면서도 시장을 독점하면서 후속 사용자가 진입하는 것을 막는 행위를 예방할 수 있다.

이 사건에서 구글은 유저 인터페이스를 재구현했고, 프로그램을 변형적으로 사용했으며, 이를 위해 필요한 만큼만 사용했다. Java의 API를 복사한 것은 법상에서 공정 이용에 해당한다.
