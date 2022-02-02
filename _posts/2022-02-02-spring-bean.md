---
title: 의존성 주입(DI)과 Spring Bean
categories:
  - spring
tags:
  - spring
  - java
---

### 태초에 디자인 패턴이 있었다.

스프링이 없는 자바 코드에서 시작해보자.

```java
public class Car {
  private MovingStrategy movingStrategy;
  private int distance;
  
  public Car(MovingStrategy movingStrategy) {
    //// private MovingStrategy movingStrategy = new RandomMovingStrategy();
    //// 일반적으로 제어권은 내부에서 생성
    this.movingStrategy = movingStrategy;
    this.distance = 0;
  }
  
  void move() {
    if (movingStrategy.isMovable()) {
      // ...
    }
  }
}

public class CarFactory {
  public static Car car() {
    return new Car(movingStrategy());
  }
  private static MovingStrategy movingStrategy() {
    return new RandomMovingStrategy();  ////
  }
}

public class Racing {
  private static final int NUM_OF_CARS;
  private List<Car> cars = new ArrayList<>();
  
  public Racing() {
    for(int i=0; i<NUM_OF_CARS: i++) {
      cars.add(CarFactory.car());
    }
  }
  
  public void move() {
    cars.forEach(Car::move);
  }
}
```

위 예는 [strategy pattern](https://youtu.be/90ZDvHl8ROE)의 전형적인 예이다. 평범하게 클래스를 만들었다면 `movingStrategy` 를 내에 생성해줬을텐데 생성자로 `movingStrategy`에 들어갈 `RandomMovingStrategy` 를 생성자에서 받아 넣어주고 있다. 그리고 위 코드의 경우는 이 또한 `CarFactory`에서 생성해주고 있다. 이 경우 이후 MovingStrategy를 변경하고 싶을 때는 `CarFactory`만 변경해주면 된다.

### Inversion of control / 의존성 주입 (DI)

이런 방식에 대해서 `제어권 역전 : Inversion of Control`이라는 표현을 사용하고 있었는데, 마틴 파울러가 [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)라는 글에서 이 표현은 애매모호하다는 의견을 제시했다. 생각해보면 '제어권의 역전'이라는 상황 자체는 UI framework나 test framework 등에서도 자주 나타나는 프레임워크의 특성이다. 위와 같은 케이스를 설명하기에는 너무 일반적인 용어이다. 이 글에서 마틴 파울러는 `의존성 주입 : Dependency Injection`이라는 용어를 사용하자고 소개한다.

> Inversion of Control
> 컨테이너들이 'Inversion of Control'을 구현했다고 말하는데, Inversion of Control은 '프레임워크'의 특성이다. 따라서 이 말은 '내 차는 바퀴가 있다는 점에서 특이해요'라는 말 밖에 되지 않는다. 문제는 '어떤 것을 역전했냐'이다.
> (중략) 이런 종류의 컨테이너들에서 역전이 일어나는 것은 plugin implementation을 보는 관점이다. (중략) 결과적으로 이러한 패턴에 대해 더 구체적인 이름이 필요하다고 생각한다. '제어권 역전'은 너무 일반적인 용어이고 그래서 많은 사람들이 헷깔려한다. 다른 많은 IoC 옹호자들과의 토론을 통해 우리는 '의존성 주입'이라는 이름에 정착하기로 했다.

### 스프링의 세계

스프링은 마틴 파울러에 글에서 '당시 IoC가 가능하다고 홍보했던 많은 컨테이너'들 중 하나이다. 처음의 코드에서 Car나 MovingStrategy가 우리가 작성할 Controller, Service와 같은 Bean 코드라고 할 수 있고, CarFactory가 Spring의 DI container라고 보면 될 것이다. 그리고 Racing과 같은 클라이언트 코드가 Spring의 내부 코드가 될 것이다. Factory에서 어떤 MovingStrategy를 쓸 지 정의하고 Car Class를 작성했던 것 처럼, 어떤 의존성을 쓸 지를 별도로 설정하고, Service나 Controller들을 작성하면 Spring의 내부 코드가 이를 읽어서 처리하고 실행한다.

> Car, MovingStrategy : Bean = Factory : Spring의 DI container = Client : Spring 내부 코드

요런 느낌...

### Bean을 등록하는 방법

* 수동 등록 

  * `@Configuration` : 이 Class는 Spring의 환경 설정과 관련되어있다고 알려줌.
  * `@Bean` : 이 어노테이션이 붙은 함수에서 리턴되는 값을 Spring Container에 Spring Bean으로 등록한다. 이 과정에서 의존성 주입도 같이 일어난다. 예를 들면 아래의 예에서는 OrderService가 등록되는데, 이는 parameter로 들어간 productRepository 인스턴스에 의존성을 갖는다.

  ```java
  @Configuration
  public class AppConfig{
    @Bean
    OrderService orderService(ProductRepository productRepository) {
      return new OrderService(productRepository);
    }
  }
  ```
  
* Annotation 붙이기. `@Component`, `@Service`, `@Controller`, `@Repository`... 등을 붙여두면 Component Scan 단계에서 Spring framework가 자동으로 인식해서 스프링빈으로 등록한다.

```java
@SpringBootApplication
public class CatApplication {

	public static void main(String[] args) {
		SpringApplication.run(CatApplication.class, args);
	}

}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

}
```

스프링 부트로 만든 어플리케이션의 main함수에 보면 위와 같이 `@SpringBootApplication` 어노테이션이 붙어있는데, 이를 따라가보면 해당 어노테이션 위에 `@ComponentScan`어노테이션이 붙은 것을 확인할 수 있다. (안의 내용은 컴포넌트 스캔에서 어떤 것을 제외할지 등에 대한 설정) 즉, main함수가 있는 class조차도 Component scan단계에서 spring bean으로 등록하고 있는 것이다.

```java
/**
 * Indicates that an annotated class is a "component".
 * Such classes are considered as candidates for auto-detection 
 * when using annotation-based configuration and classpath scanning.
 *
 * <p>Other class-level annotations may be considered as identifying
 * a component as well, typically a special kind of component:
 * e.g. the {@link Repository @Repository} annotation or AspectJ's
 * {@link org.aspectj.lang.annotation.Aspect @Aspect} annotation.
 *
 * @author Mark Fisher
 * @since 2.5
 * @see Repository
 * @see Service
 * @see Controller
 * @see org.springframework.context.annotation.ClassPathBeanDefinitionScanner
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	String value() default "";

}
```

그리고 `@Controller`나 `@Service`와 같은 어노테이션을 따라가보면 다 `@Component`를 상속받은 것을 알 수 있는데, 이 어노테이션의 주석을 보면 `Such classes are considered as candidates for auto-detection when using annotation-based configuration and classpath scanning.`라고 되어있다. 즉, 이 Component를 달고 있으면 자동으로 인식해서 bean으로 등록하게 된다.

### Annotation 방식에서 의존성을 주입하는 방법 - @Autowired

그리고 이 Annotation을 사용하는 방식에서 의존성을 주입하는 방식으로 `@Autowired` 어노테이션이 사용된다.

* 생성자에 붙이기 (Recommended!)

  ```java
  @Service
  public class OrderService {
    ProductRepository productRepository;
    
    @Autowired
    public OrderService(ProductRepository productRepository) { 
      // 생성자가 하나뿐이면 @Autowired 생략 가능
      this.productRepository = productRepository;
    }
  }
  ```

* Setter에 붙이기

  ```java
  @Service
  public class OrderService {
    ProductRepository productRepository;
    
    @Autowired
    public void setProductRepository(ProductRepository productRepository) {
      // 단점 : set method를 public으로 열어둬야 하니 변경 가능한 위험
      this.productRepository = productRepository;
    }
  }
  ```

* Field에 붙이기

* ```java
  @Service
  public class OrderService {
    @Autowired
    ProductRepository productRepository; // WARNING:: Field injection is not recommended 
    /* Spring framework 없이는 productRepository가 초기화 되지 않는 문제점이 있다. 이 때문에 Test 프레임워크 등에서는 NullpointException뜨는 등의 문제가 발생한다. 처음엔 보기에 깔끔해서 많이 썼지만 현재는 안티패턴으로 생각되고 지양되고 있어 IntelliJ도 위와 같은 워닝을 띄워준다. */
  }
  ```

* 일반 Method에 붙이기

  ```java
  public class OrderService {
    ProductRepository productRepository;
    
    @Autowired
    public void init(ProductRepository productRepository) {
      this.productRepository = productRepository;
    }
  }
  ```

생성자 방식에서 생성자가 단 하나뿐이면 `@Autowired`가 생략 가능하다고 했는데, 이를 [lombok](https://projectlombok.org/)과 함께 사용해서 편하게 주입하는 방법도 있다. lombok에서 `@RequiredArgsConstructor` Annotation을 붙이면 필수 member(final로 선언된)를 받는 생성자가 자동으로 만들어지고, 이 클래스가 Component scan의 대상인 클래스라면 단 하나의 생성자만 있기 때문에 `@Autowired`가 생략 가능해진다.  

```java
@Service
@RequiredArgsConstructor  // final이 붙거나 @NotNull 이 붙은 필드의 생성자를 자동 생성해주는 롬복 어노테이션
public class OrderService1 {
  private final ProductRepository productRepository;
}

@Service
public class OrderService2 {
  private ProductRepository productRepository;
  @Autowired
  public OrderService2(ProductRepository productRepository) {
    this.productRepository = productRepository;
  }
}
```

이 예제에서 `OrderService1`과 `OrderService2`는 동일하게 동작한다. `OrderService1` class에 `@RequiredArgsConstructor`어노테이션이 붙어있는 점과 ProductRepository에 final이 붙어있는 점을 확인하자.

### 자동으로 주입되는 Singleton 객체

그럼 여기서 궁금해지게된다. 위의 예시와 같은 경우, ProductRepository는 대체 어떻게 가져오는걸까? 수동으로 주입할 때는 직접 생성자에 인스턴스를 넣게 되므로 어떤 인스턴스가 들어갈지가 명확했다. 하지만 자동으로 주입될 객체는 어떻게 찾아오는걸까?  
스프링 컨테이너는 객체 인스턴스를 싱글톤으로 관리한다. 모든 Bean은 유일한 인스턴스로 미리 생성되어있다. (자세한 내용은 `Bean Factory`에 관련된 내용을 찾아볼 것.) 미리 생성된 Bean을 가지고 와서 의존성을 주입하게 되는 것이다. 그리고 이 과정에서 Type, 이름과 같이 Bean을 관리할 때 쓰이는 속성들을 가지고 스프링이 자체적으로 가진 우선순위에 따라 주입할 인스턴스를 찾아서 넣어주는 것이다.

```java
@Service
public class OrderService {
  ProductRepository productRepository;
  
  @Autowired
  public OrderService(ProductRepository productRepository) { 
    // 단 하나인 ProductRepository 인스턴스를 가지고 온다.
    // getBean(productRepository.class) 와 같은 느낌
    this.productRepository = productRepository;
  }
}
```

### Interface 사용시 주입되는 객체

여기서 하나가 더 궁금해진다. 위의 `ProductRepository`가 일반적인 class로 작성되었다면 해당 class를 가져올 것이라고 자연스럽게 생각할 수 있다. 그런데 만약 `ProductRepository`가 interface고 이 interface를 받아서 `DrinkRepository`와 `FoodRepository`의 두 구현 Class가 존재한다면? Spring은 어느쪽을 가져와서 OrderService에 주입할까?

```java
@Component("DrinkRepository")
public class DrinkRepository implements ProductRepository {
    public String Serve() {
        return "Serve Drink";
    }
}
@Component("FoodRepository")
public class FoodRepository implements ProductRepository {
    public String Serve() {
        return "Serve Food";
    }
}

public class ProductService {
    @Autowired
    private ProductRepository productRepository;
}
```

실제로 위와 같이 코드를 작성하면, 스프링은 ProductService 클래스의 productRepository를 생성할 때 `NoUniqueBeanDefinitionException`를 던진다. 어떤 클래스가 주입되어야 하는지 특정되지 않는다는 에러이다. 이 에러는 해결하기 위해서는 스프링 프레임워크가 내가 원하는 클래스의 인스턴스를 가져올 수 있도록 해줘야 한다. (리스트를 이용해서 해당 인터페이스를 구현한 클래스를 모두 받아오는 경우도 있지만 다른 Use case이다.) `@Autowired` 어노테이션은 의존성을 찾을 때 Type -> Qualifier -> Name 순으로 가지고 온다고 한다. Type이 Interface여서 받아오지 못한 케이스이므로, 후순위에서 결정될 수 있도록 만들어주면 된다.

```java
public class ProductService1 {
    @Autowired
    @Qualifier("FoodRepository")
    private ProductRepository productRepository;
}

public class ProductService2 {
    private ProductRepository productRepository;

    @Autowired
    public ProductService2(@Qualifier("FoodRepository") ProductRepository productRepository) {
      this.productRepository = productRepository;
    }
}
```

하나의 방법은 `@Qualifier`어노테이션으로 명시해주는 것이다. productRepository를 선언하면서 `@Qualifier` 어노테이션과 함께 어떤 클래스를 사용할건지를 명시해준다. (예시에서는 FoodRepository) 

```java
public class ProductService {
    @Autowired
    private ProductRepository foodRepository;
}
```

Qualifier도 없는 경우 필드명으로 매칭 해 해당 인스턴스를 가지고 온다고 한다. 따라서 위와 같이 필드 명을 `foodRepository`로 변경하면 `FoodRepository`로 의존성이 주입 될 것이라고 기대할 수 있다.

```java
@Component
@Primary
public class DrinkRepository implements ProductRepository {
    public String Serve() {
        return "Serve Drink";
    }
}
```

여러 구현 class 중 주로 사용되는 Class가 존재할 경우, 사용하는 측이 아니라 구현하는 쪽에서 최우선으로 사용될 Class임을 명시해주는 방법도 있다. 위의 예시와 같이 `@Primary` 어노테이션을 붙여주면 ProductRepository를 구현한 여러 Class가 있고 추가적인 Qulifier 등의 설정이 없다면 DrinkRepository가 자동으로 등록된다.

### Java의 봄과 Bean

[Spring은 Java의 겨울에 봄을 찾아 주겠다는 의미(the fact that Spring represented a fresh start after the “winter” of traditional J2EE)로 지었다고 한다.](https://spring.io/blog/2006/11/09/spring-framework-the-origins-of-a-project-and-a-name#:~:text=I%20am%20regularly%20asked%20about,of%20the%20name%20%E2%80%9CSpring.%E2%80%9D&text=Fortunately%20Yann%20stepped%20up%20with,%E2%80%9Cwinter%E2%80%9D%20of%20traditional%20J2EE.) 그런 프레임워크에서 프레임워크에 의해 만들어지는 객체들을 부르는 용어로 `Bean`이라는 단어를 채택한건 `Spring`이라는 단어만큼이나 은유적인 것 같다. 봄이 오기 전에 씨앗을 심어야 한다. 그래야 싹을 틔우고 진짜 봄을 맞이할 수 있다.

### Reference
* [[10분 테코톡] 유안의 Spring IoC/DI](https://youtu.be/_OI9mKuFb7c)
* [스프링 핵심 원리 - 기본편](https://inf.run/Xmrv)
* [Guide to Spring @Autowired](https://www.baeldung.com/spring-autowire)
* [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
* [[Spring] 의존객체 자동 주입(Automatic Dependency Injection), @Autowired, @Resource, @Inject](https://engkimbs.tistory.com/682)
* [Wiring in Spring: @Autowired, @Resource and @Inject](https://www.baeldung.com/spring-annotations-resource-inject-autowire)
