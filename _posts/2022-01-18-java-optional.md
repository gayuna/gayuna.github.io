---
title: JAVA - Optional
categories:
  - Technologies
tags:
  - java
---

이전 글에서 Generic에 대해 다뤘는데, Optional은 Java가 Generic을 사용한 또다른 예시가 되기도 함.

메서드가 반환할 결과 값이 '없음'을 명백하게 표현 할 필요가 있는데 `null`을 반환하면 에러를 유발할 가능성이 높은 상황에서 리턴 값으로 `Optional` 제네릭을 사용하자는 것이 `Optional`의 도입 목적이다.

#### Optional의 내장 함수

##### of와 ofNullable

`return Optional.of(member.getEmail());`와 같은 형태로 사용되며 argument로 받은 값을 가지는 Optional 객체를 반환한다. of는 절대로 null이 오지 않는 케이스에서만 사용되며, ofNullable은 null이 올 수도 있을 때 사용된다. null인 경우 빈 Optional 객체가 반환된다.

##### orElse

`Optional`객체가 가지고 있는 값을 꺼냅니다. 만약 빈 객체라면 인자로 받은 것을 반환합니다. `null`체크 & 리턴을 대체합니다.

```Java
if (member.isPresent()) {
    return member.get();
} else {
    return null;
}
```

``` Java
return member.orElse(null);
```

##### orElseGet

`orElse`와 비슷한 대신 인자로 함수형을 받습니다. 빈 객체일 때 해당 함수를 실행, 리턴값을 반환합니다. `orElse(...)`에서는 (...)에 해당하는 코드가 무조건 실행되기 때문에 `orElse(new ...)`와 같이 새로운 객체를 생성하거나 연산이 있는 경우라면 orElseGet을 써줘야 한다.

##### orElseThrow

`orElse`와 비슷한 대신 인자로 Exception을 생성하는 함수형을 받습니다. 빈 객체일 때 해당 함수를 실행, 생성된 예외를 던집니다.

```Java
if (member.isPresent()) {
    return member.get();
} else {
    throw new NoSuchElementException();
}
```

```Java
return member.orElseThrow(() -> new NoSuchElementException());
```

##### map, filter

`Optional`의 값이 존재한다면 map / filter가 적용해서 반환합니다.
```/* 주문을 한 회원이 살고 있는 도시를 반환한다 */
public String getCityOfMemberFromOrder(Order order) {
	return Optional.ofNullable(order)
			.map(Order::getMember)
			.map(Member::getAddress)
			.map(Address::getCity)
			.orElse("Seoul");
}
```

#### 사용시 주의사항

반환하려는 객체를 Optional로 한번 래핑한 Optional객체로 만들어서 리턴한다. Optional 객체는 `null`값을 __절대__ 가지지 않는다. `null`을 객체가 직접 가르키지 않기 위해 만들어졌는데 `null`값을 가져버리면 도입한 의미가 없어진다.

Optional을 사용하는 것은 무겁다. 따라서 아무때나 사용하지 않는다. 만약 리턴이 List라면 그냥 빈 List를 반환하는 것이 합리적이다. 

Optional을 사용하는 것은 무겁다. 따라서 쓴다면 최대한 내장 함수를 사용해주도록 한다.

Optional은 필드에 사용할 목적으로 설계되지 않았다. Class의 Member 변수로 사용하지 않는다.

Optional은 생성자나 메서드의 인자로 사용하지 않는다. Optional을 파라미터로 가지는 메서드를 만들어버리면, 해당 메서드를 호출 할 때마다 Optional을 생성해줘야 한다. 그런데 메서드 입장에서 들어오는 값에 대한 null 체크는 항상 해줘야 한다. 그렇다면 굳이 비싼 Optional을 사용하기보다 호출 되는 쪽에서 null 체크 책임을 가져가는 것이 합리적이다.

Optional을 컬렉션의 원소로 사용하지 않는다. 컬렉션에는 수많은 값이 들어가는데, 이들 하나하나를 Optional로 감싸면 cost가 높아진다. 따라서 Optional을 사용한 컬렉션을 만드는 것 보다 사용할 때 null체크를 하는 것이 합리적이다.

`Optional<T>` 대신 `OptionalInt`, `OptionalLong`, `OptionalDouble`을 사용한다.

#### Reference

[Oracle documentation의 `Class Optional<T>`](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html)
[자바8 Optional 3부: Optional을 Optional답게](https://www.daleseo.com/java8-optional-effective/)
[Java Optional 바르게 쓰기](https://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/)