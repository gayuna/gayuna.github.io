---
title: Spring batch에서 나는 error handling을 했는데 rollback only라며 에러가 날 때
categories:
  - Technologies
tags:
  - spring
  - JPA
  - spring JPA
  - Hibernate
---

#### 배경

사내에서 외부 벤더와 FTP 통신을 할 일이 자주 있어 spring batch를 많이 사용한다. 이번에 작업하던 것은 평소처럼 FTP에서 파일을 읽고 처리하는 작업이었는데, 중간에 또 다른 서비스의 API를 호출해야하는 특이점이 있었다. 문제는 이 API에서 에러가 날 수 있었다는 점이었다. 에러가 리턴되면 우리쪽의 호출부에서도 제대로 동작을 안하기 때문에 Exception이 발생했다. 나는 당연히 코드의 어느 부분에서 이를 catch해서 처리하는 부분을 만들었다. 에러가 리턴되어도 아무 문제 없고 그냥 bypass 하면 되는 상황이었기 때문에 warning 로그만 찍고 종료하도록 했다. 그런데 어라? 에러 로그가 발생한다.

```
org.springframework.orm.jpa.JpaSystemException: Transaction was marked for rollback only; cannot commit; nested exception is org.hibernate.TransactionException: Transaction was marked for rollback only; cannot commit
```

#### 원인 분석

가장 이해가 안되는 부분은 에러 핸들링을 했는데 이러한 종류의 에러가 난다는 것이었다. [Stack Overflow의 답변](https://stackoverflow.com/a/19311268)에서 이유를 찾을 수 있었다.

When you mark your method as @Transactional, occurrence of any exception inside your method will mark the surrounding TX as roll-back only (__even if you catch them__). You can use other attributes of @Transactional annotation to prevent it of rolling back like:

에러를 잡았나 여부와는 상관 없이 Transactional안에서 Exception이 발생하면 roll-back only로 마킹한다는 것이다. 

Tasklet Step을 사용했고, 하나의 Tasklet은 하나의 Transaction으로 감싸져있다. 내부에서 사용되는 메소드들에 @Transaction이 붙어있었지만 모두 기본 설정이거나 거기에 readonly 옵션만 붙어있었기 때문에 propagation의 설정이 REQUIRED였을 것이다. 따라서 새로 트랜잭션을 열지 않고 batch의 step에서 열린 트랜잭션을 그대로 가지고 들어간다. 그리고 이 상태에서 Exception이 발생하면 내가 그것을 핸들링을 했더라도 이미 해당 트랜잭션이 roll-back only로 마킹되어있기 때문에 step이 종료되면서 커밋되는 순간에 roll-back only라며 에러를 뱉어내던 것이다.

#### 해결 방법

##### 방법 1. 내부 트랜잭션을 분리해서 가장 상위의 transaction이 커밋되도록 한다.

`@Transactional(propagation = Propagation.REQUIRES_NEW)`로 새로운 트랜잭션을 생성해서 메소드에 진입하도록 할 수 있다. nested 옵션을 쓰고싶었으나 `JpaDialect does not support savepoints - check your JPA provider's capabilities` 라는 에러와 함께 사용이 되지 않았다. [이건 사용하는 DB가 지원을 해야하는 부분이다.](https://techblog.woowahan.com/2606/)

##### 방법 2. 사용자 정의 Exception을 throw 하도록 하고 이를 rollback 대상에서 제외한다.

API를 call 한 결과로 Exception을 직접 Throw 하고 있었기 때문에 이렇게 우회할 수 있었다. 기존에 IllegalStateException을 던지고 있었는데, 이를 상속받아 예외처리할 전용 Exception을 만들었다. 그리고 `@Transactional(noRollbackFor = [MyNewException::class])`와 같이 어노테이션에서 해당 익셉션을 롤백 마킹에서 제외하도록 했다.

더 나은 방안이 있을지는 고민중...

* https://stackoverflow.com/a/19311268
* https://godekdls.github.io/Spring%20Batch/configuringastep/#52-taskletstep
* https://techblog.woowahan.com/2606/
