---
title: JAVA - Generic
categories:
  - Technologies
tags:
  - java
---

[Oracle의 자바 튜토리얼 페이지](https://docs.oracle.com/javase/tutorial/java/generics/index.html)의 일부를 발췌하여 설명을 더하거나 빼고 쓴 글입니다.

#### 내가 만드는 제네릭 : 제네릭 타입

generic class는 `class name<T1, T2, ..., Tn> { /* ... */ }`과 같은 형식으로 정의합니다.
class 이름 다음에 <>안에 type parameter(T1, T2...)들이 옵니다. 이 자리에는 어떤 type, class, interface, 심지어는 다른 사용자 정의 type까지 상관 없이 들어올 수 있습니다.
관용적으로 type parameter는 대문자 한글자가 오는데, 아래와 같은 글자가 많이 쓰입니다.

* E - Element (used extensively by the Java Collections Framework)
* K - Key
* N - Number
* T - Type
* V - Value
* S,U,V etc. - 2nd, 3rd, 4th types

예시로 타입 T를 담을 수 있는 상자 class를 만들어보도록 하겠습니다.

```java
public class Cart<T> {
    // T stands for "Type"
    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

이 Cart는 `Cart<Bread> breadCart = new Cart<Bread>();`와 같은 방식으로 인스턴스로 만들어 사용할 수 있습니다 Java SE 7 이후로는 `Cart<Bread> breadCart = new Cart<>();`와 같이 써도 됩니다.

#### 내가 만드는 제네릭 : 제네릭 메서드

제네릭 메서드를 만드는 것은 제네릭 타입을 만드는 것과 비슷합니다. 단, type parameter의 범위는 메서드 안으로 제한됩니다. static 함수와 non-static 함수를 모두 만들 수 있으며 generic class의 생성자도 만들 수 있습니다.

```java
public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
               p1.getValue().equals(p2.getValue());
    }
}

public class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey()   { return key; }
    public V getValue() { return value; }
}
```

이 예제에서 제네릭 타입 `Pair<K, V>`의 생성자 `public Pair(K key, V value)`와 Util클래스의 `public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2)` 메서드는 둘 다 제네릭 메서드입니다. 이 둘을 통해 어떤 타입의 key든 받아 어떤 타입이든 value를 저장할 수 있는 Pair class와 생성된 Pair 인스턴스를 비교할 수 있는 compare 메서드를 만들 수 있습니다. compare 함수의 리턴 타입 앞에 `<K, V>`가 써져있는 것을 확인하세요.

#### Type Parameter에 제한 걸기

Type Parameter에 특정 타입을 상속받은 타입들만 올 수 있도록 제한을 할 수 있습니다. 우리의 카트 class가 앞으로 신선 식품만 담을 수 있게 제한해보겠습니다. 

```java
public class Cart<T extends FreshProduct> {
    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

Parameter는 여러개의 제한을 가질 수 있습니다. 이 때는 `<T extends B1 & B2 & B3>`와 같은 형태로 사용합니다. 이 때 B1, B2, B3 의 순서는 class가 먼저, interface가 나중에 와야합니다. 만약 interface가 먼저 오고 나서 class가 온다면 컴파일 에러가 발생합니다.

제네릭 메서드에서도 Type parameter를 한정시킬 수 있습니다. 이는 타입에 관계 없이 사용할 수 있는 generic method를 만들 때 핵심이 됩니다. 배열 `anArray`에서 특정 값 `elem`보다 더 큰 요소의 갯수를 세는 함수를 만든다고 가정해보겠습니다. 아래와 같이 loop를 돌면서 각 요소들과 elem을 비교해야하는데, 그냥 제네릭으로만 정의하면 비교연산 `>`이 가능한지가 보장되지 않기 때문에 컴파일에러가 발생합니다.

```Java
public static <T> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e > elem)  // compiler error
            ++count;
    return count;
}
```

비교 연산이 가능한 타입만 T로 들어올 수 있도록 하기 위해 `Comparable<T>`인터페이스로 제한을 하면 컴파일 에러가 사라집니다.

```Java
public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e.compareTo(elem) > 0)
            ++count;
    return count;
}
```

#### 제네릭의 와일드카드

제네릭에서 물음표 `?`는 와일드카드로 사용됩니다. 와일드카드를 parameter, field, 지역 변수, 리턴 타입 등 다양한 곳에 사용할 수 있습니다. The wildcard is never used as a type argument for a generic method invocation, a generic class instance creation, or a supertype.

##### Upper bounded wildcard

위에서 Cart가 담을 수 있는 것을 신선 상품으로 한정했었는데요, Upper Bounded Wildcard는 비슷한 동작을 하게 만듭니다. `public static void process(List<? extends Foo> list) { /* ... */ }`라는 함수가 있다고 가정해보면, process 함수는 Foo나 Foo를 상속받는 class를 요소로 가지는 리스트를 인자로 받습니다. 그래서 `for (Foo elem : list) { /* ... */ }`와 같이 사용할 수 있습니다. 예를 들어 `public static double sumOfList(List<? extends Number> list)`라는 함수는 숫자를 상속받는 타입을 인자로 받아서 모두 더해 double을 리턴하는 함수입니다. 자연수, 정수, 실수 상관 없이 받아서 double로 리턴할 수 있게 됩니다.

##### Unbounded wildcard

unbounded wildcard는

* Object class에 있는 것 만으로 메서드를 만들 때
* type parameter와 관계 없는 것만으로 메서드를 만들 때

사용합니다.

예를들어 아래와 같은 메서드는 리스트 안에 어떤 타입이 들어있든 메서드가 해당 타입에 관계 없이 도착하기 때문에 unbounded wildcard를 사용하게 됩니다. 비슷한 경우로 List의 `size`나 `clear` 메서드가 있습니다.

```Java
public static void printList(List<?> list) {
    for (Object elem: list)
        System.out.print(elem + " ");
    System.out.println();
}
```

##### Lower bounded wildcard

Lower bounded wildcard는 upper bounded wildcard와 반대로 동작합니다. `<? super A>`와 같은 형태로 특정 class가 상속받은 base class들만 들어오게 한정할 수 있습니다. 여기서 주의 할 것은 upper bounded wildcard와 lower bounded wildcard는 동시에 사용할 수 없다는 것입니다. 생각해보면 당연합니다. 두개가 동시에 적용이 된다면 결국 특정 타입 하나만 쓰는 것과 다르지 않을테니까요.

#### 제한

제네릭을 만들 때의 제한 사항들.

* Primitive type으로 Generic Type을 만들 수 없다. : 예를 들어 `Pair<int, char>`는 불가. `Pair<Integer, Character>`는 가능
* Type Parameter로 인스턴스를 생성할 수 없다. 예를들어 `public static <E> void apend(List<E> list)` 라는 함수 안에서 `E elem = new E();`와 같이 사용할 수 없다.
* Type parameter로 static field를 정의할 수 없다.
* Parameterized 된 type들을 캐스팅하거나 `instanceof`를 사용할 수 없다.
* Parameterized 된 type들로 Array를 만들 수 없다. : 예를 들어 `List<Integer>[] arrayOfLists = new List<Integer>[2];`와 같이 사용할 수 없다. 이렇게 쓸 수 있다면 `stringLists[1] = new ArrayList<Integer>();`와 같이 활용할 수 있게 되는데 강력한 type check가 목적인 generic과도 Java와도 어울리지 않는다.
* Parameterized 된 type으로 Exception을 만들 수 없다. : 예를 들어 `class MathException<T> extends Exception { /* ... */ }`와 같이 사용할 수 없다.
* Generic 상태에서 Parameter Type이 같은 함수들은 오버로딩이 불가능하다. : 예를들어 같은 클래스 안에 `public void print(Set<String> strSet)`와 `public void print(Set<Integer> intSet)`를 동시에 정의할 수 없다. type eraser가 일어난 후에 둘의 파라미터가 `Set<Object>`로 동일하기 때문에 구분할 수 없다.

#### Todo

* 사용 이유 옮기기
* Erasure
* Restrictions 더 길게
