#### 제네릭 메서드로 만들라.

매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다.\
예) Collections의 알고리즘 메서드(binarySearch, sort 등)

```java
public class Item30 {

    public static Set union(Set s1, Set s2) {
        Set result = new HashSet(s1);
        result.addAll(s2);
        return result;
    }
}
```
경고
```java
Warning:(9, 22) java: unchecked call to HashSet(java.util.Collection<? extends E>) as a member of the raw type java.util.HashSet

Warning:(10, 22) java: unchecked call to addAll(java.util.Collection<? extends E>) as a member of the raw type java.util.Set
```

메서드 선언에서 세집합(입력2개, 반환1개)의 원소 타입을 타입 매개변수로 명시하고 메서드안에서도 이 타입 매개변수만 사용하게 수정한다.\
(타입 매개변수들을 선언하는) 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.\
타입매개변수 목록은 \<E>\
반환타입은 Set\<E>

```java
public static <E> Set<E> unionGeneric(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}

public static void main(String[] args) {
    Set<String> guys = Set.of("lee", "kim", "park");
    Set<String> mans = Set.of("choi", "na", "hwang");
    Set<String> union = unionGeneric(guys, mans);
    System.out.println(union);
}
```
결과
```java
[choi, na, hwang, lee, kim, park]
```

집합 3개(입력 2개, 반환 1개)의 타입이 모두 같아야 한다.\
이를 한정적 와일드카드 타입을 사용하여 더 유연하게 개선할 수 있다.

불변객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다.\
제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.\
하지만 이렇게 하려면 요청한 타입 매개 변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다.\
제네릭 싱글턴 팩터리라고 하며, Collections.reverseOrder와 같은 함수 객체나 Collections.emptySet 같은 컬렉션용으로 사용한다.

```java
@SuppressWarnings("unchecked")
public static <T> Comparator<T> reverseOrder() {
    return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
}

@SuppressWarnings("unchecked")
public static final <T> Set<T> emptySet() {
    return (Set<T>) EMPTY_SET;
}
```

항등함수(identity function)를 담은 클래스

항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것은 낭비다.\
자바의 제네릭이 실체화된다면 항등함수를 타입별로 하나씩 만들어야 했겠지만, 소거 방식을 사용하던 덕에 제네릭 싱글턴 하나면 충분하다.

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T>UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}

public static void main(String[] args) {
    String[] strings = {"삼베", "대마", "나일론"};
    UnaryOperator<String> sameString = identityFunction();
    for (String s : strings) {
        System.out.println(sameString.apply(s));
    }

    Number[] numbers = {1, 2.0, 3L };
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number n : numbers) {
        System.out.println(sameNumber.apply(n));
    }
}
```
IDENTITY_FN을 UnaryOperator<T>로 형변환하면 비검사 형변환 경고가 발생한다.\
T가 어떤 타입이든 UnaryOperator<Object>는 UnaryOperator<T>가 아니기 때문이다.\
하지만 항등함수란 입력값을 수정없이 그대로 반환하는 특별한 함수로 T가 어떤 타입이든 UnaryOperator<T>를 사용해도 타입 안전하다.\
이 사실을 알고 있으니 이 메서드가 내보내는 비검사 형변환 경고를 숨겨도 안심할 수 있다.\
@SuppressWarnings("unchecked") 애너테이션을 추가하면 오류나 경고없이 컴파일된다.

제네릭 싱글턴을 UnaryOperator<String>, UnaryOperator<Number>로 형변환을 하지 않아도 컴파일 오류나 경고가 발생하지 않는다.

자기 자신이 들어간 표현식을 사용하여 타입매개변수의 허용 범위를 한정할 수 있다.\
재귀적 타입 한정(recursive type bound)라는 개념이다.\
재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰인다.

```java
public interface Comparable<T> {
  public int compareTo(T o);
}
```
타입매개변수 T는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.\
실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다.\
따라서 String은 Comparable<String>을 구현하고 Integer는 Comparable<Integer>를 구현하는식이다.

Comparable을 구현한 원소의 컬렉션을 입력받은 메서드들은 주로 그 원소들을 정렬 혹은 검색하거나, 최소값, 최대값을 구하는 식으로 사용된다.

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어있습니다.");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

        return  result;
}

public static void main(String[] args) {
  List<Integer> list = Arrays.asList(1, 2, 5, 3);
  System.out.println(max(list));
}
```
타입 한정인 \<E extends Comparable\<E>>는 '모든 타입 E는 자신과 비교할 수 있다'라고 읽을 수 있다.(상호 비교 가능)\
이 메서드는 빈 컬렉션을 건네면 IllegalArgumentException을 던지니, Optional\<E>를 반환하도록 고치는 편이 나을 것이다.

재귀적 타입 한정은 훨씬 복잡해질 가능성이 있긴 하지만, 다행히 그런일은 잘 일어나지 않는다.\
관용구, 와일드카드를 사용한 변경, 시뮬레이트한 셀프 타입 관용구를 이해하고 나면 실전에서 마주치는 대부분의 재귀적 타입 한정을 무리없이 다를 수 있을 것이다.

_핵심정리_

> - 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드 보다 제네릭 메서드가 더 안전하며 사용하기 쉽다.
> - 메서드도 형변환없이 사용할 수 있는 편이 좋으며, 제네릭 메서드가 되어야 한다.
> - 형변환을 해줘야 하는 기존 메서드는 제네릭하게 만든다.
> - 기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을 훨씬 편하게 만들어줄 것이다.

[참조] (이펙티브 자바 3판)
