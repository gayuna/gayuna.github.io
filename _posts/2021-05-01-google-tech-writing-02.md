---
title: Google technical writing One, Pre-class material 읽기 (2/3)
categories:
  - Translation
tags:
  - Technical Writing
---

[구글 테크니컬 라이팅 페이지](https://developers.google.com/tech-writing)의 하위 페이지인 [pre-class material](https://developers.google.com/tech-writing/one)의 중간 부분. 원래 상하로만 나누려고 했는데 너무 길어진다... 말투도 앞과 바뀌었다. 3편까지 쓰고 전체적인 퇴고 하면서 톤도 맞추고 오타도 고치고 스타일도 정리 해야겠다.

## List and tables

### Choose the correct type of list

 테크니컬 라이팅에는 아래와 같은 리스트가 주로 사용됩니다.

* 불렛 리스트
* 숫자 리스트
* 임베드디드 리스트

순서가 없는 리스트에는 불렛 리스트를 사용하세요; 순서가 있는 리스트에는 숫자 리스트를 사용하세요. 즉,

* 불렛 리스트에 있는 항목들의 순서를 바꾸면 의미가 바뀌지 않습니다.
* 숫자 리스트에 있는 항목들의 순서를 바꾸면 의미가 바뀝니다.

예를들어, 아래 예시에서는 순서를 바꿔도 의미가 바뀌지 않기 때문에 불렛 리스트를 사용합니다.

```md
Bash provides the following string manipulation mechanisms:

* deleting a substring from the start of a string
* reading an entire file into one string variable
```

반면 아래 리스트는 숫서를 바꾸면 의미가 바뀌기 때문에 숫자 리스트가 되어야만 합니다.

```md
Take the following steps to reconfigure the server:

1. Stop the server.
2. Edit the configuration file.
3. Restart the server.
```

임베디드 리스트 (run-in list라고도 불림)는 문장 안에 항목들을 가지고 있습니다. 예를들어, 다음 문장은 4개 항목을 가진 임베드 리스트를 가지고 있습니다.

```md
The llamacatcher API enables callers to create and query llamas, analyze alpacas, delete vicugnas, and track dromedaries.
```

일반적으로 임베드 리스트는 기술 문서를 표현하는데 좋지 않은 방법입니다. 불렛 리스트나 숫자 리스트로 바꿔보세요. 예를들어 위 문장은 아래와 같이 바꿀 수 있습니다.

```md
The llamacatcher API enables callers to do the following:

* Create and query llamas.
* Analyze alpacas.
* Delete vicugnas.
* Track dromedaries.
```

### Keep list items parallel

좋은 리스트와 좋지 않은 리스트는 무엇이 다를까요? 좋은 리스트는 각 항목이 동등하고 좋지 않은 리스트는 동등하지 않습니다. 항목들이 동등하면 한데 묶이는 것처럼 보입니다. 다시 말해 동등한 리스트는 다음과 같은 요소들이 동일합니다.

* grammar
* logical category
* capitalization
* punctuation

반대로, 동등하지 않은 리스트는 이 중에 하나가 다릅니다.

예를들어 아래 리스트는 모두 복수를 사용하고, 먹을 수 있으며, 소문자로 쓰여졌고, 점이나 반점등의 문장부호가 사용되지 않았기 때문에 동등합니다.

* carrots
* potatoes
* cabbages

하지만 아래 리스트는 동등하지 않습니다.

* carrots
* potatoes
* The summer light obscures all memories of winter.

다음 문장은 모든 항목들이 완전한 문장으로 구성되어 있으며 첫 글자에 대문자를 사용하고 마지막에 마침표를 사용했다는 점에서 동등합니다.

* Carrots contain lots of Vitamin A.
* Potatoes taste delicious.
* Cabbages provide oodles of Vitamin K.

### Start numbered list items with imperative verbs

숫자 리스트의 시작을 open, start같은 명령어로 시작하세요.

1. Download the Frambus app from Google Play or iTunes.
2. Configure the Frambus app's settings.
3. Start the Frambus app.

### Punctuate items approriately

리스트의 항목이 문장이라면, 첫 글자를 대문자로 만들고 마침표를 사용하고 그렇지 않은 경우에는 대문자나 마침표를 사용하지 말세요. 예를들어 아래 문장에서는 M을 대문자로 쓰고 마침표도 었습니다.

* Most carambolas have five ridges.

하지만 아래 문장에서는 t를 소문자로 쓰고 마침표도 찍지 않습니다.

* the color of lemons

### Create useful tables

분석적인 사람은 표를 좋아합니다. 한 페이지에 문단이 여러개 있고 표가 하나뿐이라면 엔지니어는 표에 집중합니다.

표를 만들 때는 다음 가이드라인을 참고하세요.

* 각 열을 의미있는 헤더로 라벨링하라. 독자가 각 열이 무슨 의미인지 추측하게 하지 말라.
* 표 칸에 너무 많은 텍스트를 넣지 말라. 한 셀에 두 문장 이상 넣어야 한다면 다른 포맷을 사용해야 하지 않은지 생각해라.
* 각 열은 다른 데이터를 담겠지만 각 열 안에서는 같은 타입의 데이터를 갖도록 해라. 즉 같은 열에 숫자와 코끼리를 함께 담으려고 하면 안된다.

### Introduce each list and table

각 리스트나 표를 어떤 정보를 나타내는지 알려주는 문장과 함께 소개하는 것이 좋습니다. 리스트나 표에도 문맥을 주는 것이죠. 이 소개하는 문장은 온점보다 콜론으로 끝내도록 합니다.
필수적인 것은 아니지만, 이 문장에는 __following__ 이라는 단어를 넣는 것이 좋습니다. 예를 들면 다음과 같은 문장입니다:

```text
The following list identifies key performance parameters:

Take the following steps to install the Frambus package:

The following table summarizes our product's features against our key competitors' features:
```

## Paragraphs

파트의 주제별로 글의 종속성을 정리하고 이 파트들을 논리적인 흐름에 따라 전개해 독자가 이해할 수 있도록 합니다.

### Write great opening sentence

시작하는 문장은 모든 문단에서 가장 중요합니다. 바쁜 독자들은 첫 문장만 읽고 따라오는 문장은 넘어가기도 합니다. 따라서 여러분의 글씨기 여력을 첫번째 문장에 집중해야 합니다.
좋은 시작하는 문장은 문단의 중점을 만듭니다. 예를들어 다음 문단은 효과적인 시작 문장을 보여줍니다:

```text
A loop runs the same block of code multiple times. For example, suppose you wrote a block of code that detected whether an input line ended with a period. To evaluate a million input lines, create a loop that runs a million times.
```

처음에 오는 시작하는 문장이 문단의 주제를 루프를 소개하는 것이라고 알려줍니다. 반면에 다음 문단의 시작하는 문장은 독자들을 헷깔리게 합니다.

```text
A block of code is any set of contiguous code within the same function. For example, suppose you wrote a block of code that detected whether an input line ended with a period. To evaluate a million input lines, create a loop that runs a million times.
```

### Focus each paragraph on a single topic

하나의 문단은 하나의 독립적인 논리 단위를 표현해야 합니다. 각 문단이 현재 주제에서 벗어나지 않게 하세요. 이전 주제나 미래의 주제에 대해 서술하게 하지 마세요. 퇴고할 때 현재 주제와 맞지 않는다면 꼭 지우거나 다른 문단으로 옮기세요.

예를들어, 다음 문단에서 첫 문장이 주제를 나타내고 있다면 어떤 문장을 지워야 할까요?

```md
The Pythagorean Theorem states that the sum of the squares of both legs of a right triangle is equal to the square of the hypotenuse. The perimeter of a triangle is equal to the sum of the three sides. You can use the Pythagorean Theorem to measure diagonal distances. For example, if you know the length and width of a ping-pong table, you can use the Pythagorean Theorem to determine the diagonal distance. To calculate the perimeter of the ping-pong table, sum the length and the width, and then multiply that sum by 2.
```

2번째, 5번째 문장을 제거함으로써 피타고라스의 정리에 관해서만 다루도록 해야합니다.

```markdown
The Pythagorean Theorem states that the sum of the squares of both legs of a right triangle is equal to the square of the hypotenuse. You can use the Pythagorean Theorem to measure diagonal distances. For example, if you know the length and width of a ping-pong table, you can use the Pythagorean Theorem to determine the diagonal distance.
```

### Don’t make paragraphs too long or too short

긴문단은 시각적으로 접근하기 어렵습니다. 너무 긴 문단은 글자의 벽으로 보이고 독자들은 무시합니다. 독자들은 일반적으로 3~5문장으로 구성된 문단을 선호하지요. 7문장이 넘어간다면 읽고싶지 않아합니다. 퇴고할 때 너무 긴 문단은 두개의 문단으로 구분해보면 좋습니다.
반대로, 너무 짧은 문단도 만들면 안됩니다. 만약 문서가 문장 1개짜리 문단으로 가득하다면 구성에 문제가 있는 것입니다. 문장 한개짜리 문단을 여러 문장의 문단으로 묶거나 리스트로 바꾸는 방법을 찾아야 합니다.

### Answer what why and how

좋은 문단은 아래 세 질문에 대한 답을 가지고 있습니다:

1. 독자에게 어떤 것을 말하고 싶은가?
1. 왜 독자에게 이것을 말하고 싶은가?
1. 독자들이 이 지식을 어떻게 사용해야 하는가? 혹은 독자가 어떤 행동을 취해야 하는가?

예를 들어 다음 문단은 what,why, how에 대해 답하고 있습니다.

## Audience

좋은 문서화 = 일을 하기 위해 필요한 지식과 스킬 - 현재 지닌 지식과 스킬

다른 말로 독자에게 필요하지만 아직 알지 못하는 정보는 문서가 제공해줘야 합니다. 이 절에서는 다음을 어떻게 하는지 살펴봅니다.

* 독자 정의하기
* 독자가 배워야 하는 것 정하기
* 독자에 맞게 문서화하기

### Define your audience

중요한 문서화는 독자를 정의하는데 상당한 시간과 노력을 들입니다. 이런 노력에는 설문, 사용자 경험 연구, focus groups 그리고 문서화 테스트가 있습니다. 그만큼의 시간은 없는당신을 위해 이 절에서는 간단한 접근을 사용합니다.

독자의 정의를 그들의 역할로 시작해보세요. 예를들면:

* 소프트웨어 엔지니어
* 기술적필드의, 비엔지니어 (예를 들면 테크니컬 프로그램 매니저)
* 과학자
* 과학 분야의 전문가들 (예를 들면 의사)
* 공학 학부생
* 공학 대학원생
* 비 기술 직군

비 기술직군이라도 기술적, 수학적 기술이 있는 사람이라면 기쁘겠지만 우리는 일단 독자를 정의할 때는 이런 역할을 최우선순위에 두어야 합니다. 같은 역할을 수행하는 사람은 일반적으로 같은 기본 지식과 기술을 가집니다. 예를 들면 :

* 대부분의 소프트웨어 엔지니어는 유명한 정렬 알고리즘, big O 표기법, 그리고 하나 이상의 프로그래밍 언어를 알고 있습니다. 따라서 소프으퉤어 엔지니어는 O(n)을 알거라 가정할 수 있지만 비 기술 직군에겐 그렇게 할 수 없습니다.
* 의사를 대상으로 하는 연구 리포트는 같은 연구에 대해 일반 대중을 대상으로 쓰여진 기사와는 매우 다릅니다.
* 새로운 머신 러닝 접근에 대해 대학원생에게 설명할 때와 학부 1학년에게 설명할 때 교수님의 설명은 달라야 합니다.

같은 역할을 수행하는 사람들 모두가 정확히 같은 지식을 가지고 있다면 글은 훨씬 쓰기 쉬울 것입니다. 불행하게도 같은 역할을 가지고 있더라도 지식은 매우 차이가 납니다. Amal은 파이썬을 잘하고 Sharon은 C++전문가이며 Micah는 Java를 잘하고 Kara는 리눅스를 사랑하지만 David는 iOS밖에 모르는 사람이에요.

역할만으로 독자를 정의하기엔 부족합니다. 당신의 예상 독자의 예상 지식 수준을 알고 있어야 합니다. Frombus 프로젝트의 소프트웨어 엔지니어는 연관된 프로젝트인 Dingus에 대해서는 아는 바가 있겠지만 전혀 상관 없는 Carambola에 대해서는 전혀 모를 것입니다. 평균적으로 심장 전문가는 귀의 문제에 대해 평균적인 소프트웨어 엔지니어보다 잘 알겠지만 청각학자보다는 훨씬 아는 바가 없을 것입니다.

시간 또한 영향을 줍니다. 대부분의 소프트웨어 엔지니어는 미적분학을 배웠겠지만 일하는데는 그것을 사용하지 않기 대문에 미적분학에 대한 지식은 점점 희미해집니다. 반대로 신입 엔지니어에 비해 경력이 있는 사람들은 그들의 현재 프로젝트에 대해 훨씬 잘 알고 있습니다.

### Determine what your audience needs to learn

목표를 달성하기 위해 당신의 독자가 배워야 하는 모든 것을 쓰세요. 어떤 경우에는 이 리스트는 예상 독자가 행해야 하는 과제를 포함하기도 합니다. 예를 들어 :

```text
After reading the documentation, the audience will know how to do the following tasks:
```

```text
Use the Zylmon API to list hotels by price.
Use the Zylmon API to list hotels by location.
Use the Zylmon API to list hotels by user ratings.
```

예상 독자가 특정 순서로 과제를 해야하는 경우도 있음에 유의하세요. 예를들어 어떤 종류의 프로그램을 새로 쓰기 위한 지식을 배우기 전에 어떻게 빌드하고 실행해야 하는지를 먼저 배워야할 수 있습니다.

디자인 스펙을 쓰는 경우에 이 리스트는 예상 독자가 해야 하는 행동보다 알아야 하는 정보에 집중해야 합니다. 예를 들면:

```text
After reading the design spec, the audience will learn the following:
```

```text
Three reasons why Zylmon outperforms Zyljeune.
Five reasons why Zylmon consumed 5.25 engineering years to develop.
```

### Fit documentation to your audience

독자의 니즈를 맞추면서 글을 쓰려면 타인을 배려하는 공감 능력이 필요합니다. 스스로의 궁금증보다는 독자의 궁금증을 해결해주는 설명을 써야 합니다. 어떻게 하면 스스로가 아닌 독자를 위한 글을 쓸 수 있을까요? 거기에 대한 쉬운 답은 없지만 집중해야 할 몇가지 지표를 제시할 수 있습니다.

#### Vocabulary and concepts

단어 수준을 독자에 맞추세요. Words 챕터를 참고하세요.

근접성의 문제를 항상 기억하세요. 같은 팀의 사람은 팀 내의 약어를 이해하겠지만 외부의 인간이 같은 약어를 이해할 수 있을까요? 독자의 범위가 넓어질수록 더 많이 설명해야 합니다.

또한 당신의 팀을 경험한 사람이라면 당신의 팀 프로젝트의 구현 방법이나 데이터 구조에 대해 잘 이해하겠지만 그 외의 사람들 (팀에 새로온 사람을 포함하여)은 그렇지 않습니다. 팀에 기존에 있던 멤버들만은 위해 쓰는 글이 아니라면, 당신이 생각하는 것보다 더 많이 써야 합니다.

#### Curse of knowledge

전문가들은 종종 '지식의 저주'를 겪습니다. 이는 그들이 주제에 관해 잘 이해하고 있다 보니 새로 진입하는 사람들에게 설명을 잘 못해주는 상황입니다. 전문가들은 자신이 이미 알고있는 것을 초보자가 알지 못한다는 것을 까먹을 때가 많습니다. 초보자들은 전문가들이 자세히 설명해주지 않은 미묘한 상호작용과 깊은 시스템을 참조하는 설명을 이해하지 못할 수 있습니다.

초보자의 시선에서 지식의 저주는 아직 컴파일 되지 않은 모듈에 의한 "File not found" 링크 에러와 같습니다.

#### Simple words

영어는 세계적으로 기술적인 커뮤니케이션을 하는데 가장 많이 사용되는 언어가 되었습니다. 하지만 대부분의 독자는 영어가 아닌 다른 언어에서 더 편안함을 느깁니다. 따라서 복잡한 단어보다 단순한 단어를 사용하도록 하세요; 오래되거나 너무 복잡한 영어 단어는 피하세요. 엄청나게 긴 단어나 자주 사용하지 않는 단어는 몇몇 독자를 몰아낼 수 있습니다.

#### Cultural neutrality and idioms

당신의 글이 문화적으로 중립되게 하세요. 읽는 사람이 소프트웨어를 파악하기 위해 나스카, 크리켓 혹은 스모의 복잡함에 대해 알아야 하도록 만들지 마세요. 예를들어 다음 문장은 야구 관련 은유로 가득 차있어서 파리 사람에겐 혼돈을 줄 수 있습니다:

```text
If Frambus 5.0 was a solid single, Frambus 6.0 is a stand-up double.
```

관용구는 구로서의 의미가 단어 하나하나의 의미가 다릅니다. 예를들어 다음 구절들이 관용구입니다 :

* a piece of cake
* Bob's your uncle

Cake? Bob? 대부분의 미국 독자들은 첫번재 관용구를 이해할 것이고 대부분의 영국 독자들은 두번째 관용구를 이해할 것입니다. 당신이 독자를 영국인으로만 한정한다면 두번째 구를 사용해도 되겠지요. 하지만 당신이 세계의 독자들을 상대로 글을 쓴다면 이를 대체하도록 하세요.

관용구는 우리가 사용하는 말에 뿌리 깊게 박혀있어서 문자 그대로의 의미는 잘 느껴지지 않습니다. 즉, 관용구는 또 다른 형태의 '지식의 저주'입니다.

글을 읽게 되는 사람 중 일부는 번역기를 사용한다는 것을 생각하세요. 변역기들은 평서문의 간단한 영어에 비해 문화적 레퍼런스나 관용구에서 작동이 잘 안되는 경향이 있습니다.
