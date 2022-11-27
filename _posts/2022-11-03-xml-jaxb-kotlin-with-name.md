---
title: Kotlin data class에서 JAXB로 XML파일 생성 시 Element명을 필드명과 다르게 만들기
categories:
  - tinytip
tags:
  - Kotlin
  - JAXB
  - XML
  - Annotation
---

고객사 스펙에 맞추느라 XML파일을 만들어서 FTP로 업로드해야 하는 일이 생겼다. 사내 다른 레포지토리에서 XML을 다루는 부분을 참고해서 파일을 생성하고 파싱하려고 했다. 최초로 만들었던 코드는 아래와 같았다.

```kotlin
@XmlAccessorType(XmlAccessType.FIELD)
@XmlRootElement(name = "Message")
data class MyDto(
    @XmlElement(name = "Id")
    private val id:String? = null,
    /*JAXB가 no-arg constructor를 필요로 해서 모든 필드를 nullable로 만들었다.
    사실 파싱 부분은 이해가 되어도 생성 부분에서는 필요 없는게 맞을 것 같은데,
    찾아봐도 디자인이 나쁜 것 같다 라는 코멘트들은 있지만 여전히 no-args constructor를 필요로 하는 것 같다.*/
    @XmlElement(name = "Data")
    private val data: List<MyDataDto>? = null
) {}
```

이 때 의도를 했던 것은 각 필드들을 가지고 XML을 만들되, 이름을 필드명이 아닌 `@XmlElement` 어노테이션에 지정된 name을 사용하여 맨 앞이 대문자가 되도록 하고 싶었다. (고객 스펙) 하지만 테스트를 해보니 XML이 잘 생성되긴 하지만 맨 앞이 대문자가 만들어지지 않았다. 필드명이 그대로 사용되고 있는 것 같았다.

구글링을 해서 비슷한 경우들을 찾아보았다. [거기 달린 답변을 보니](https://stackoverflow.com/a/60961309/7701097) 일단 XmlAccessType이 달랐다. `XmlAccessType`의 설명을 읽어보니 `XmlAccessType.FIELD`의 경우는 필드들을 기본적으로 다 XML에 생성하고, `XmlAccessType.NONE`의 경우는 따로 어노테이션이 붙어있는 경우만 생성한다는 것 같았다. 이미 어노테이션을 달아뒀기때문에 NONE으로 바꾼 후 다시 테스트를 해보았다. 가장 상위의 Message만 남고 나머지가 다 사라지는 현상을 볼 수 있었다. 분명 Annotation을 붙였는데 어째서?

해당 답변에서는 세가지 중 하나를 하라고 했지만 그것만으로 해결되지 않았기 때문에 다른 조건들도 살펴보았다. `Move the @XmlElement(name = "Security") annotation down to the getSecurities() method`라는 문장이 눈에 띄고, 실제 예시에도 모두 get 함수가 만들어져있었다. 모든 필드를 private에서 public으로 변경해보았다. 또한 코틀린 어노테이션에는 사용 지점 대상을 지정할 수 있는 기능이 있다. 이를 이용해서 getter에만 해당 어노테이션을 지정해서 테스트해보았다.

```kotlin
@XmlAccessorType(XmlAccessType.NONE)
@XmlRootElement(name = "Message")
data class MyDto(
    @get:XmlElement(name = "Id")
    val id:String? = null,
    @get:XmlElement(name = "Data")
    val data: List<MyDataDto>? = null
) {}
```

이번엔 잘 만들어진다. 예상컨데 XmlAccessType가 field인 경우는 getter를 사용하지 않고 필드 값들을 그대로 읽어와서 XML파일을 생성하는 것 같고 (그래서 필드명이 그대로 사용됨) 나머지 경우에는 getter를 사용해서 값을 읽어서 파일을 만들고 그 와중에 XmlElement어노테이션이 사용되는 것 같다. (반대로 아마 XML -> object로 파싱할 때는 @set을 붙여주었다.)
