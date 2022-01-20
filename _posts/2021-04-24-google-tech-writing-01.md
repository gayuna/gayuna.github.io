---
title: Google technical writing One, Pre-class material 읽기 (1/3)
excerpt_separator: "<!--more-->"
categories:
  - Translation
tags:
  - Technical Writing
---

[구글 테크니컬 라이팅 페이지](https://developers.google.com/tech-writing)에서는 최근 온라인 강의도 제공한다. 지난 달에 체크했을 때는 한국 시간으로 평일 낮 2시 이래서 도저히 신청 할 엄두를 내지 못했는데, 이번에 뜬 일정들이 4월 26일 8AM PT(27일 0시)랑 5월 4일 1PM CEST(4일 20시)여서 둘 중 하나는 들을 수 있겠다 싶어서 다시 [pre-class material](https://developers.google.com/tech-writing/one)을 읽었다. 형식적으로도, 단어 사용의 측면에서도 이런 글이 있다는 것 자체가 너무 재밌고 도음도 많이 됐었다. (물론 영어로 쓰여졌기 때문에 주로 영어로 쓸 때 도움이 되지만... 업무에서는 커밋메세지부터 모두 한글로 쓰고 있기 때문에.) 특징이라면 우리가 일반적으로 말하는 '영어를 잘하는 법'이나 '영어 시험 잘보는 법'과는 전혀 접근이 다르다. 오히려 정반대인 부분도 많다.

## Words

### Define new or unfamiliar terms

용어가 이미 존재한다면 좋은 설명을 링크한다.
문서가 용어를 소개한다면, 용어를 정의. 여러 용어를 소개한다면 정의들을 glossary에 모은다.

### Useterms consistently

함수 중간에 변수명을 바꾸면 컴파일이 안되듯, 문서 중간에 용어의 이름을 바꾸면 독자의 머릿속에서 당신의 아이디어가 컴파일 되지 않을 것이다.

__문서 내내 하나의 용어에 대해 같은 단어를 사용할 것__. 아래 문장에서 Protocol Buffers를 protobufs로 중간에 이름을 바꾸고 있음. (안좋은 예)

```markdown
_Protocol Bufffers_ provide their own definition language. Blah, blah, blah. And that’s why _protobufs_ have won so many county fairs.
```

쓰고싶다면 긴 정식 이름을 먼저 사용하면서 약어도 함께 쓴 다음 문서 내내 약어로 표현할 것.

```markdown
_Protocol buffers (or protobufs for short)_ provide their own definition language. Blah, blah, blah. And that’s why _protobufs_ have won so many county fairs.
```

### Use Acronyms properly

약어를 쓸 때는 __정식 이름을 먼저 쓰고 괄호로 약어를 소개__ 하도록 한다. 이 때 __풀네임과 약어를 모두 굵게 처리__ 한다.

```markdown
This document is for engineers who are new to the __Telekinetic Tactile Network (TTN)__ or need to understand how to order TTN replacement parts through finger motions.
```

### Use the acronym or the full term?

약어는 추상화일 뿐이고, 개발자들은 줄임말을 만나면 읽으면서 머릿속에서 이를 확장해야 하기 때문에 다 써두는 것보다 시간을 더 소비한다. 단, 충분히 익숙해지고 나면 더이상 이런 작업을 하지 않는다. 많은 웹 개발자가 HTML이 무었의 약자인지 잊었을 것이다.

* 몇번만 사용된다면 약어로 정의하지 말 것
* 아래 두가지 조건이 충족 되는 약어를 정의할 것
  * Full therm에 비해 길이가 확연히 줄어들 때
  * 문서 안에서 여러번 반복적으로 등장할 때

### Disambiguate pronouns

프로그래밍에서 포인터가 에러를 불러일으키곤 하듯이 대명사는 곧잘 문제를 발생시킨다. 대명사를 제대로 사용하지 않는 것은 마치 독자의 머리속에서 null pointer error를 발생시킨다. __많은 경우에서 대명사를 쓰는 것보다 명사를 반복해서 쓰는 것이 낫지만__, 대명사의 유용성이 위험도를 앞지르는 경우도 있다.

다음과 같은 대명사 가이드라인을 확인하라.

* 대명사는 명사가 사용 된 _후에_사용할 것. 명사 사용 전에 대명사 사용하는 것은 금지
* 대명사와 명사를 가깝게 유지할 것. 5단어 이상 떨어진다면 대명사 대신 명사를 반복할 것을 생각할 것.
* 명사와 대명사 사이에 또 다른 명사가 있다면 대명사를 사용하지 말고 명사를 반복할 것

### It They

아래 대명사들이 기술 문서에서 가장 많은 혼란을 일으킨다.

* It
* They, them, their

### This and that

아래 대명사들도 문제가 잘 일어난다.

* This
* That

This/ that의 모호함을 피하기 위해 다음과 같은 전략을 사용하라

* This, that을 적당한 명사로 대체하기.
* This, that 바로뒤에 명샤를 배치하기.

`Overlapping functionality is not optimal` ->
`This overlapping functionality is not optimal`

## Active voice

### Prefer active voice to passive voice

되도록 능동태 문장을 쓸 것. 수동태는 자제할 것. 능동태로 쓴 문장은 다음과 같은 장점을 가짐.

* 대부분의 독자는 수동태를 머릿속에서 능동태로 고친다. 독자에게 추가적으로 처리가 필요한 글을 쓸 이유가 없다. 능동태로 쓴다면 독자들은 프리프로세싱을 하지 않고 바로 컴파일할 수 있게 되는 것이다.
* 수동태는 행동을 간접적으로 전달하여 글의 요지가 헷깔리게 만든다.
* 몇몇 수동채 문장들은 주체를 다 생략해서 독자가 추측하게 만든다.
* 능동태 문장이 일반적으로 수동태 문장보다 짧다.

Be bold—be active.

## Clear sentences

코메디 작가들은 재미있는 글을 쓰고 호러 작가는 무서운 글을 쓰는 것처럼 테크니컬 라이터들은 명확한 글을 쓴다.

### Choose strong verbs

문장에서 동사가 가장 중요하고, 동사를 정확히 고르면 문장 전체가 명확해진다. 항상 적확한, 강한, 구체적인 동사를 사용하려고 노력할 것. 아래와 같은 적확하지 않고, 약하며 일반적인 단어들은 피할 것.

* be동사: is, are, am, was, were, etc.
* occur
* happen

|Weak Verb|Strong Verb|
|------|---|
|The error __occurs__ when clicking the Submit button.|Clicking the Submit button __triggers__ the error.|
|This error message __happens__ when...|The system __generates__ this error message when...|
|We __are__ very careful to ensure...|We carefully __ensure__...|

be동사만 너무 많이 쓰고 있을 수 있다. 줄이자.

일반적인 단어가 너무 많다는 것은 다음과 같은 다른 문제의 신호일 수 있다.

* 적확하지 않거나 문장의 주체가 보이지 않는 경우
* 수동태가 사용되었을 경우

### Reduce there is/there are

There is나 There are로 시작하는 문장은 일반 명사와 일반 동사의 결합. 이런 조합보다 실질 명사와 실질 동사를 사용할 것.

이상적인 경우, 단순히 There is, There are를 지우기만 하면 된다.

`There is a variable called met_trick that stores the current accuracy.`

There is를 없앤 아래 두 문장은 위 문장보다 명확하다.

`A variable named met_trick stores the current accuracy.`

`The met_trick variable stores the current accuracy.`

문장의 뒤에 있는 실질 주어, 실질 동사를 앞으로 끌고 와서 There is나 There are를 제거할 수도 있다. 다음 문장에서 대명사 you가 문장의 끝에 나오는걸 확인하자.

`There are two disturbing facts about Perl you should know.`

There are를 you로 대체함으로서 문장이 강화된다.

`You should know two disturbing facts about Perl.`

또 다른 상황으로 문장에 진짜 주어나 동사가 존재하지 않는 경우가 있다. 진짜 주어가 없다면 하나 만들어주자. 다음 문장에서는 받는 주체가 정의되어있지 않다.

`There is no guarantee that the updates will be received in sequential order.`

There is를 좀 더 의미있는 주어 (예를 들면 클라이언트)로 대체하면 독자가 더 명확하게 이해할 수 있다.

`Clients might not receive the updates in sequential order.`

### Minimize certain adjectives and adverbs (optional)

형용사와 부사들은 시와 소설에서는 글을 풍부하게 하지만 정확하게 정의되지 않은 경우가 많고 독자에게 주관적인 경우가 많아 기술적 문서에는 적합하지 않다. 심지어 기술 문서가 아니라 마케팅을 위한 위험한 글인 것처럼 보일 수도 있다. 기술 문서에 이런 문장이 있다고 상상해보자.

`Setting this flag makes the application run screamingly fast.`

엄청나게 빠르다는 표현은 독자의 주의를 끌겠지만 좋은 방향일지는 모른다. 이보다는 독자에게 실제 데이터를 제공하자. 형용사와 부사 대신 다음과 같이 객관적인 수치로 된 정보를 제공하자.

`Setting this flag makes the application run 225-250% faster.`

## Short sentences

개발자들은 다음과 같은 이유로 코드 라인 수를 줄이려고 한다.

* 코드가 짧으면 다른 사람이 읽기 더 좋다.
* 코드가 짧으면 코드의 유지보수가 더 용이하다.
* 코드가 길어지면 그 부분의 코드에서 문제가 날 확률이 높다.

기술 문서를 쓸 때도 같은 법칙이 적용된다.

* 짧은 문서는 긴 문서보다 읽기 쉽다.
* 짧은 문서는 긴 문서보다 유지보수하기 좋다.
* 문서가 길어지면 그 부분에서 문제가 날 확률이 높다.

문서를 짧게 쓰는건 시간이 더 오래 걸리지만 결과적으로 그럴만한 가치가 있다. 짧은 문장들은 긴 문장보다 소통에 용이하다. 일반적으로 짧은 문장은 긴 문장보다 이해하기 쉽다.

### Focus each sentence on a single idea

한 문장에는 하나의 아이디어, 생각, 컨셉만 담도록 한다. 프로그램의 하나의 명령문이 하나의 작업만 수행하는 것처럼 하나의 문장은 하나의 아이디어만 수행한다. 아래와 같은 긴 문장은 여러개의 생각을 담고 있다.

```text
The late 1950s was a key era for programming languages because IBM introduced Fortran in 1957 and John McCarthy introduced Lisp the following year, which gave programmers both an iterative way of solving problems and a recursive way.
```

긴 문장을 한문장의 하나에 아이디어만 담은 문장의 연속으로 쪼갠다면 다음과 같은 결과를 얻을 수 있다.

```text
The late 1950s was a key era for programming languages. IBM introduced Fortran in 1957. John McCarthy invented Lisp the following year. Consequently, by the late 1950s, programmers could solve problems iteratively or recursively.
```

### Convert some long sentences to lists

기술 문서의 긴 문장은 리스트로 만들 수 있는 경우가 많다. 아래 문장을 예로 들면

```text
To alter the usual flow of a loop, you may use either a break statement (which hops you out of the current loop) or a continue statement (which skips past the remainder of the current iteration of the current loop).
```

긴 문장에 접속사 or가 포함되어 있다면 불렛 리스트로 바꾸는 것을 고려해보면 좋다. 긴 문장 안에 리스트가 있거나 작업 과정이 포함되어 있다면 불렛 리스트나 숫자를 붙인 리스트를 고려하자. 예를들면 위 문장에는 or가 있으니 불렛 리스트로 바꿔보자.

```markdown
To alter the usual flow of a loop, call one of the following statements:  

* break, which hops you out of the current loop.
* continue, which skips past the remainder of the current iteration of the current loop.
```

### Eliminate or reduce extraneous words

많은 문장들은 독자에게 정보는 주지 않으면서 공간을 차지 하는 필러들을 포함하고 있다. 예를 들면 다음 문장에서 필수적이지 않은 단어들을 찾을 수 있다.

```text
An input value greater than 100 causes the triggering of logging.
```

`causes the triggering of`라는 긴 구문을 짧은 `triggers`로 대체하면 짧은 문장으로 쓸 수 있다.

```text
An input value greater than 100 triggers logging.
```

아래 문장은 긴 구절을 동사 하나로 바꿀 수 있다.

```text
This design document provides a detailed description of Project Frambus.
```

```text
This design document describes Project Frambus.
```

아래와 같이 자주 쓰이는 구를 단어로 바꿀 수 있다.

|Wordy|Concise|
|------|---|
|at this point in time|now|
|determine the location of|find|
|is able to|can|

### Reduce subordinate clauses (optional)

절은 문장 안에 있는 주체와 동작을 포함하는 독립적인 논리 구문이다. 모든 문장은

* 주절
* 0개 혹은 하나 이상의 종속절

을 포함한다. 종속적을 주절의 아이디어를 수정한다. 이름에서 느껴지듯 종속절은 주절보다 덜 중요하다. 다음 문장을 예로 보자.

```markdown
Python is an interpreted programming language, which was invented in 1991.

* 주절: Python is an interpreted programming language
* 종속절: which was invented in 1991
```

종속절은 일반적으로 앞에 오는 접속사를 기준으로 할 수 있다. 다음은 종속절을 가지고 오는 접속사들의 예시이다.

* which
* that
* because
* whose
* until
* unless
* since

어떤 종속절은 반점과 함께 시작되지만 그렇지 않은 경우도 있다. 다음 문장에서 종속절은 because와 함께 시작하지만 반점은 보이지 않는다.

```text
I prefer to code in C++ because I like strong data typing.
```

글을 쓸 때 종속절에 주의를 기울이자. 한 문장 = 하나의 아이디어를 가슴에 새기자. 종속절이 하나의 아이디어를 확장시키고 있는가 아니면 다른 아이디어로 분기하고 있는가? 만약 후자라면 종속절(들)을 별개의 문장으로 만들자.

### Distinguish that from which

That과 which는 둘 다 종속적을 가져온다. 어떤 국가에선 두개의 차이가 별로 없기도 하지만 미국에서는 두개가 구분된다. 미국에서 which는 중요하지 않은 종속절에 사용되고 that은 생략되면 안되는 중요한 종속절에 사용된다. 예를들어 다음 문장의 메세지는 파이썬은 인터프리터 언어라는 사실이고 귀도 반 로섬이 만들었다는 부분이 없어도 성립된다.

```text
Python is an interpreted language, which Guido van Rossum invented.
```

반면 다음 문장은 선형대수를 포함하지 않는다는 내용을 꼭 필요로 한다.

```text
Fortran is perfect for mathematical calculations that don't involve linear algebra.
```

문장을 읽어보고 종속절 전에 한번 끊게 된다면 which를 쓰고 그렇지 않다면 that을 쓰자. 그리고 which 앞에 반점을 삽입하자.
