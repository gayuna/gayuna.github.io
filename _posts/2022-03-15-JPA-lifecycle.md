---
title: Spring JPA의 영속성와 Proxy
categories:
  - Technologies
tags:
  - spring
  - JPA
  - spring JPA
  - Hibernate
---

* 세미나용 정리.
* 자바 ORM 표준 JPA 프로그래밍의 3장, 8장 내용을 많이 가져왔음을 밝힙니다.

### 엔티티 매니저 팩토리 / 엔티티 메니저 / 트랜잭션

JPA를 시작할 때 엔티티 매니저 팩토리를 생성한다.

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
/*persistence.xml에서 이름이 jpabook인 영속성 유닛을 찾아서 엔티티 매니저 팩토리를 생성.

이 때 JPA를 동작시키기 위한 기본적인 객체를 만들거나 DB 커넥션 풀도 만든다. (JPA 구현체에 따라 다름) -> EMF를 생성하는 비용은 매우 크다 -> Application에서 한번 만들고 공유해서 사용한다.*/

EntityManager em = emf.createEntityManager();
/*
엔티티 매니저 팩토리에서 엔티티 매니저를 생성한다. JPA의 기능 대부분은 엔티티 매니저가 제공한다. (DB에 엔티티를 등록/수정/삭제/조회) 엔티티는 내부에 데이터 소스(DB connection)를 유지하면서 데이터베이스와 통신한다. DB connection과 관계가 있기 때문에 스레드간에 공유하거나 재사용하면 안된다.
*/
EntityTransaction tx = em.getTransaction(); // 트랜젝션 API

try{ /*JPA를 사용할 때 항상 트랜잭션 안에서 데이터를 변경해야 한다.*/
    tx.begin(); // 트랜잭션 시작. 보통 이 시점에 entity manager가 connection을 획득함.
    logic(em); // 비지니스 로직 (em.persist em.find등)
    tx.commit(); // 트랜잭션 커밋
} catch (Exception e) {
    tx.rollback(); // 예외 발생시 트랜잭션 롤백
}

em.close(); // 사용이 끝난 엔티티 매니저 종료
emf.close(); // 어플리케이션 종료 시 엔티티 매니저 팩토리 종료

```

![1](https://media.vlpt.us/images/cham/post/aa30cb4a-9c40-4d1e-aed9-500a2548689e/image.png)

### 영속성 컨텍스트와 엔티티의 생명 주기

엔티티를 영구 저장하는 환경. 엔티티 매니저로 엔티티를 저장/조회시에 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관/관리한다. 예를들어 `em.persist(member);`를 할 때 엔티티 매니저를 사용해서 회원 엔티티를 영속성 컨텍스트에 저장하는 것. 영속성 컨텍스트는 엔티티 매니절르 생성할 때 하나 만들어진다.

![3](https://media.geeksforgeeks.org/wp-content/uploads/20210626212614/GFGHibernateLifecycle.png)

엔티티는 영속성 컨텍스트의 관계에 따라 4가지 상태를 가진다.

#### 비영속 (Transient State) : 영속성 컨텍스트와 관계가 없는 상태

엔티티 객체를 생성했으나 순수한 객체 상태이고 아직 아무것도 저장하지 않은 상태. 따라서 영속성 컨텍스트나 DB와는 관련이 없다.

#### 영속 (Persistent State): 영속성 컨텍스트에 저장된 상태

엔티티 매니저를 통해 엔티티가 영속성 컨텍스트에 저장된 상태. 이제 엔티티는 영속성 컨텍스트에 의해 관리된다. 직접 저장했을 때 뿐만 아니라, get을 통해 DB에서 엔티티를 읽어 온 경우도 영속 상태로 리턴되게 된다.

#### 준영속 (Detached State): 영속성 컨텍스트에 저장되었다가 분리된 상태

영속성 컨텍스트가 엔티티를 관리하다가 관리하지 않게 된 경우. `em.detach()`를 호출해서 수동으로 닫거나, `em.close()`를 통해 영속성 컨텍스트 자체를 닫거나 `em.clear()`를 통해 영속성 컨텍스트를 초기화 하는 방법 등을 통해 이 상태로 전이된다.

#### 삭제 (Removed State): 삭제된 상태

데이터를 영속성 컨텍스트와 _데이터 베이스에서 삭제_ 한다.

### 영속성 컨텍스트와 함께하는 쓰기 / 읽기 / 수정 / 삭제

#### 쓰기

![5](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FblsBzW%2FbtqFcN1dVRK%2FEYpjNc8R8tnJSpRtIC6Fj1%2Fimg.png)
![9](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FPoi51%2FbtqFaDfbl3g%2FLk3LB6966XcEG8nuNdyXC0%2Fimg.png)
* 엔티티가 영속 상태로 되었을 때 먼저 1차 캐시에 데이터가 저장된다.
* 영속성 컨텍스트는 엔티티를 식별자 값(`@Id`)로 구분 -> 영속 상태에서는 반드시 식별자값이 있어야한다.
* JPA의 경우 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 DB에 반영 : `플러시(Flush)`
* 엔티티 매니저는 트랜잭션을 커밋하기 전까지 DB에 엔티티를 저장하지 않고 내부 쿼리 저장소에 INSERT SQL을 모아둔다. 그리고 트랜잭션을 커밋할 때 모아둔 커리를 한번에 DB에 보낸다. (쓰기 지연)

#### 읽기 

![7](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbgRAP6%2FbtqFbB1uCQG%2F6yRnxmE2SevKVNMWMsLAK1%2Fimg.png)

* Id가 member1인 엔티티를 조회한다면 영속성 컨텍스트는 1차 캐시에서 이를 찾아 리턴한다.
* 만약 Id가 1차 캐시에 없다면 그 때 DB에서 조회 해 1차 캐시에 저장한 후 리턴한다.

#### 수정

![11](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdG1iTX%2FbtqFa7GzKP5%2FtCELwKVoc1dz9DgjNN58CK%2Fimg.png)

* `memberA.setName("John")`과 같은 코드를 실행하면 1차캐시의 Entity에 값이 저장된다.
* 커밋 하는 순간 플러시가 실행되는데, 이 때 현재 엔티티와 엔티티의 스냅샷을 비교해서 변경된 엔티티를 찾는다.
* 변경된 엔티티가 있으면 수정 쿼리를 생성해서 쓰기 지연 SQL 저장소에 UPDATE 쿼리문을 보낸다. -> 이게 DB에 날라간다.

#### 삭제

* `em.remove(memberA)`와 같이 remove에 삭제할 엔티티를 넘겨주면 삭제 쿼리를 쓰기 지연 SQL 저장소에 등록한다. 다음 Flush 때 실제로 DB에 DELETE 쿼리가 나가면서 삭제된다.
* remove를 호출 한 순간 memberA는 영속성 컨텍스트에서 제외된다. 이렇게 삭제된 엔티티는 재사용하지 말고 GC되도록 두는 것이 좋다.

### 프록시 객체

엔티티를 조회할 대 연관된 엔티티들이 모두 조회되어야 하는 것은 아니다. 예를 들어 다음과 같은 코드가 있다고 하자.

```java
@Entity
public class Member {
    private String username;
    @ManyToOne
    private Team team;
    public Team getTeam() {
        return team;
    }
    public String getUsername() {
        return username;
    }
}

@Entity
public class Team {
    private String name;
    public String getName() {
        return name;
    }
}
```

회원 정보를 조회하고 싶다고 가정했을 때, 만약 팀 이름까지 조회해야 한다면 Team entity도 읽어야겠지만 회원 이름만 필요하다면 굳이 Team Entity까지는 불러오지 않아도 된다.
JPA애서는 엔티티가 실제 사용될 때까지 DB 조회를 지연하는 방법을 제공하는데 이것을 `지연 로딩`이라고 한다. 이 경우 `memberA.getTeam()`을 할 때 Team 엔티티의 내용을 조회한다.
이 때 DB조회를 지연하기 위해 사용되는 가짜 객체를 프록시 객체라고 한다.
프록시 클래스는 실제 클래스를 상속 받아서 만들어지므로 실제 클래스와 겉으로 보기에 같다. 사용할 때 프록시 객체인지 신경쓰지 않고 사용해도 된다. 프록시 객체는 실제 객체에 대한 참조인 `target`을 보관한다.

### 프록시 객체의 초기화

![13](https://miro.medium.com/max/1400/1*oqkczABnxAnbvlGjVdmJSA.png)

1. 프록시 객체에 getName()이 호출 되었을 때 (위 코드에서는 Team의 getName, 그림에서는 Member의 getName) 실제 데이터가 있는지 조회한다. 
1. 실제 엔티티가 생성되어 있지 않다면 프록시 객체는 영속성 컨텍스트에 실제 엔티티 생성을 요청한다.
1. 영속성 컨텍스트는 데이터 베이스를 조회해서 실제 엔티티 객체를 생성한다.
1. 프록시 객체는 실제 엔티티 객체의 참조를 target에 보관하고 이 엔티티 객체의 getName을 호출해서 결과를 반환한다.

* 프록시 객체를 초기화 한다고 프록시 객체가 엔티티로 바뀌는게 아니라 실제 엔티티에 접근하는 것임에 유의한다.
* 프록시 객체는 상속받은 class이므로 타입 체크시에 유의.
* 프록시 객체의 초기화는 영속성 컨텍스트를 통해 이루어지므로 준영속 상태의 프록시를 초기화하면 문제가 발생한다. (Hibernate의 경우 `org.hibernate.LazyInitializationException`)

### 즉시 로딩과 지연 로딩

엔티티를 조회할 때 연관된 엔티티도 함께 조회하는 경우를 `즉시 로딩`이라 하고 `@ManyToOne(fetch = FetchType.EAGER)`와 같은 방법으로 설정한다. 한편 연관된 엔티티는 실제 사용할 때 조회하는 방법을 `지연 로딩`이라고 하고 `@ManyToOne(fetch = FetchType.LAZY)`와 같은 방법으로 설정한다.

#### 즉시 로딩과 JOIN

즉시 로딩을 할 때 두개 엔티티 정보를 가져와야 하는데, 성능 최적화를 위해 대부분의 JPA 구현체는 쿼리를 두번 보내는 대신 JOIN 쿼리를 사용한다. 이 때 JPA의 경우 기본적으로 외부 조인(LEFT OUTER JOIN)을 사용한다. 위의 Member - Team 코드에서 Member 안의 Team에 null이 들어갈 수 있었다. 내부 조인을 사용하게 되면 어느 팀에도 소속되지 않은 회원을 조회할 수 없게 되기 때문에 외부 조인을 사용한것이다. _모든 회원이 하나의 팀에 소속되어 있다고 제한한다면_ 즉 team이 null이 되지 않는다면 내부 조인을 사용해서 성능을 끌어올릴 수 있다. `@JoinColumn(name="TEAM_ID", nullable = false)`와 같이 nullable을 false로 만들거나 `@ManyToOne(fetch = FetchType.EAGER, optional=false)`와 같이 @ManyToOne의 Optional 설정을 False로 만들면 내부 조인을 사용하게 된다.

#### 지연로딩과 프록시 객체

지연로딩의 프록시 객체는 DB에 접근하는 비용을 줄이는 것이다. 그런데 만약 이미 영속성 컨텍스트에 해당 엔티티가 올라와있다면 굳이 프록시 객체를 사용할 이유가 없다. 따라서 이 경우에는 프록시 객체를 사용하지 않고 실제 엔티티 객체를 사용한다.

### Hibernate에서 Collection의 로딩과 JPA의 기본 Fetch 전략

이번엔 Member가 여러 팀에 동시에 소속될 수 있다고 가정해보자. `List<Team> teams`와 같이 컬렉션이 사용되었을 것이다. 하이버네이트는 엔티티를 영속 상태로 만들 때 (Member 엔티티) 엔티티에 컬렉션이 있다면 (List<Team>) 원본 컬렉션을 하이버네이트가 제공하는 내장 컬렉션으로 변경한다. 이를 `컬렉션 래퍼`라고 한다.
그리고 이런 컬렉션의 지연로딩은 프록시 객체가 아닌 이 컬렉션 레퍼가 수행한다. `memberA.getTeams()` 와 같이 컬렉션에 접근 할 때까지는 초기화하지 않고, `memberA.getTeams().get(0)`과 같이 실제로 데이터에 접근할 때 DB를 조회해서 초기화한다.

그리고 JPA의 기본 Fetch 전략은 __객체는 즉시로딩 컬렉션은 지연로딩__ 이다. 즉 `@ManyToOne`, `@oneToOne`과 같은 경우는 즉시 로딩, `@OneToMany`, `@ManyToMany`와 같은 경우는 지연 로딩이 된다.

### 영속성 전이

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶다면 영속성 전이(transitive persistence)기능을 사용하면 된다. JPA에서는 `CASCADE` 옵션으로 설정할 수 있다. 예를들어 위의 Member와 Team의 코드에서 각각의 엔티티는 별도로 저장해야 한다. (Member 엔티티와 Team 엔티티 각각 em.persist를 해줘야 한다는 뜻.)

```java
@Entity
public class Parent {
    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy="parent")
    private List<Child> children = new ArrayList<Child>();
}

@Entity
public class Child {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Parent parent;
}
```

예를들어 위와 같은 엔티티들이 있다고 가정했을 때, Parent 클래스의 객체 parentA의 Children에 childA, childB가 있다고 한다면 세개의 객체에 모두 연관관계 매핑을 해준 후 세번 각각 em.persist를 호출해 영속 상태로 만들어줘야 한다.

이 때 `@OneToMany(cascade = CascadeType.PERSIST)`와 같이 설정하면 `persist`시에 부모와 자식 엔티티를 한번에 영속화할 수 있다. (em.persist(parent)만 하면 child도 동시에 영속화.) 다만, 이는 영속화만 도와줄 뿐 연관관계 매핑과는 관련이 없기 때문에 영속화 전에 양방향으로 연관관계의 추가가 먼저 이루어져야 한다.

반대로 삭제의 경우에도 `cascade = CascadeType.REMOVE`와 같이 설정하면 부모 엔티티가 삭제되었을 때 자식 엔티티도 함께 삭제되도록 할 수 있다. cascade 옵션은 `cascade = {CascadeType.PERSIST, CascadeType.REMOVE}`와 같이 중괄호를 이용해 여러개를 동시에 설정할 수 있다. PERSIST, REMOVE의 전이는 함수 호출 시점이 아니라 Flush 호출 시점임에 유의하도록 한다.

```java
public enum CascadeType {
    ALL,
    PERSIST,
    MERGE,
    REMOVE,
    REFRESH,
    DETACH
}
```
