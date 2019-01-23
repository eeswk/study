#### 로 타입(raw type, 무인자 자료형)은 사용하지 말라

> 제네릭 클래스 or 제네릭 인터페이스(제네릭 타입)란 ?
> 클래스와 인터페이스 선언에 타입 매개변수(type parameter)를 사용

제네릭 타입은 일련의 매개변수화 타입(parameterized type)을 정의한다.

```java
List<String>
```
원소의 타입이 String인 리스트를 뜻하는 매겨변수화 타입.
String이 정규(formal) 타입 매개변수 E에 해당하는 실제(actual) 타입 매개변수다.

제네릭 타입을 하나 정의하면 그에 딸린 로 타입(raw type)도 함께 정의된다.
로 타입이란?
제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.
List\<E>의 로 타입은 List다.

제네릭이 지원하기 전에 컬렉션 사용시
```java
// Stamp 인스턴스만 취급한다.
private final Collection stamps = ...;

//실수로 동전을 넣는다.
stamps.add(new Coin(...));

for (Iterator i = stamps.iterator(); i.hasNext();) {
  Stamp stamp = (Stamp) i.next(); //ClassCastException 발생
  stamp.cancel();
}
```
컬렉션에서 이 동전을 꺼내기 전에는 오류를 알아채지 못한다.
오류는 컴파일 할때 발견하는 것이 좋다.
이 경우, 런타임에야 알 수 있게 된다.

제네릭사용
```java
// Stamp 인스턴스만 취급한다.
// 매개변수화된 컬렉션타입 - 타입 안정성 확보
private final Collection<Stamp> stamps = ...;

//실수로 동전을 넣는다.
stamps.add(new Coin(...)); //컴파일 오류

//형변환 불필요
for (Stamp s : stamps) {
  //...
}

```
**로 타입(raw type)을 쓰면 제네릭이 안겨주는 안정상과 표현력을 모두 잃게 된다.**

List 같은 로 타입은 사용해서는 안되나, List\<Object> 처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다.

List와 List\<Object> 차이점?

List는 제네릭 타입에서 완전히 발을 뺀것.\
List\<Object>는 모든 타입을 허용

```java
package effect.item26;

import java.util.ArrayList;
import java.util.List;

public class Item26 {
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(43));
        String s = strings.get(0);  // 컴파일러가 자동으로 형변환 코드를 넣어준다.
    }
    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }
}
```
결과 실행시 오류
```java

Exception in thread "main" java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String (java.lang.Integer and java.lang.String are in module java.base of loader 'bootstrap')
	at effect.item26.Item26.main(Item26.java:10)

```

매개변수화 타입인 List <Object> 로 변경시

```java
package effect.item26;

import java.util.ArrayList;
import java.util.List;

public class Item26 {
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(43));
        String s = strings.get(0);  // 컴파일러가 자동으로 형변환 코드를 넣어준다.
    }
    private static void unsafeAdd(List<Object> list, Object o) {
        list.add(o);
    }
}
```
결과 컴파일시 오류
```java
Error:(9, 19) java: incompatible types: java.util.List<java.lang.String> cannot be converted to java.util.List<java.lang.Object>
```

```java
//모르는 타입의 원소도 받는 로 타입을 사용
static int numElementsInCommon(Set s1, Set s2) {
    int result = 0;
    for (Object o1 : s1)
        if (s2.contains(o1))
            result++;
    return result;
}
```
이 메서드는 동작은 하지만 로 타입을 사용해 안전하지 않다.

그렇다면, 비한정적 와일드카드 타입(unbounded wildcard type)을 대신 사용하는게 좋다.
제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않다면 물음표(?)를 사용한다.
제네릭 타입인 Set\<E>의 비한정적 와일드 카드 타입은 Set\<?>이다.
어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 Set 타입이다.

```java
//비한정적 와일드카드 타입을 사용
static int numElementsInCommons(Set<?> s1, Set<?> s2) {
    int result = 0;
    for (Object o1 : s1)
        if (s2.contains(o1))
            result++;
    return result;
}
```
와일드카드 타입은 안전하고, 로 타입은 안전하지 않다.
로 타입 컬렉션에는 아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉽다.
Collection<?>에는 (null 외에는) 어떤 원소도 넣을 수 있다.
다른 원소를 넣으려 하면 컴파일할 때 오류 메시지가 발생한다.
```java
// Stamp 인스턴스만 취급한다.
// 와일드카드 타입 사용
private final Collection<?> stamps = ...;

//실수로 동전을 넣는다.
stamps.add(new Coin(...)); //컴파일 오류
```
결과
```java
Error:(43, 20) java: incompatible types: effect.item26.Coin cannot be converted to capture#1 of ?
```

즉, 컬렉션의 타입 불변식을 훼손하지 못하게 막았다.
구체적으로는, (null 외의) 어떤 원소도 Collection<?>에 넣지 못하게 했으며, 컬렉션에서 커낼 수 있는 객체의 타입도 전혀 알 수 없게 했다.
이러한 제약을 받아들일 수 없다면 제네릭 메서드나 한정적 와일드 카드 타입을 사용하면된다.

로 타입을 쓰지말라는 규칙에도 예외가 몇개 있다.
**class 리터럴에는 로 타입을 써야 한다.**
자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다.(배열과 기본 타입은 허용한다)

List.class, String[].class, int.class는 허용하고 List\<String>.class와 List<?>.class는 허용하지 않는다.

instanceof 연산자와 관련
런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다.
그리고 로 타입이든 비한정적 와일드카드 타입이든 instanceof는 완전히 똑같이 동작한다.
비한정적 와일드카드 타입의 꺾쇠괄호와 물음표는 아무런 역할이 없이 코드만 지저분하게 만드므로, 차라리 로타입을 쓰는 편이 깔끔하다.
제네릭 타입에 instanceof를 사용하는 예

```java
if (o instanceof Set) { //로 타입
  Set<?> s = (Set<?>) o;  //와일드카드 타입
}
```
o 타입이 Set임을 확인한 다음 와일드카드 타입인 Set<?>로 형변환해야 한다.
(로 타입인 Set이 아니다).
이는 검사 형변환(checked cast)이므로 컴파일러 경고가 뜨지 않는다.

_핵심정리_
> - 로 타입을 사용하면 런타임 예외가 일어날 수 있으니 사용하면 안된다.
> - 로타입은 제네릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐이다.
> - Set\<Object>는 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입이고,
> - Set<?>는 모종의 타입 객체만 저장할 수 있는 와일드 카드 타입이다.
> - Set (로 타입)은 제네릭 타입 시스템에 속하지 않는다.
> - Set\<Object>와 Set<?>은 안전하지만 Set은 안전하지 않다.
---

용어정리

한글용어|영문용어|예
--------|--------|--------
매개변수화 타입 | parameterized type |List<String>
실제 타입 매개변수| actual type parameter | String
제네릭 타입 | generic type | Lis\<E>
정규타입 매개 타입 | formal type parameter | E
비한정적 와일드카드 타입 | unbounded wildcard type | List<?>
로 타입 | raw type | List
한정적 타입 매개변수 | bounded type parameter | \<E extends Number>
재귀적 타입 한정| recursive type bound | <T extends Comparable<T>>
한정적 와일드카드 타입| bounded wildcard type | List<? extends Number>
제네릭 메서드| generic method | static \<E> List\<E> asList(E[] a)
타입 토큰 | type token |String.class

[참조] (이펙티브 자바 3판)
