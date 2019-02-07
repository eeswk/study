#### 한정적 와일드카드를 사용해 API 유연성을 높이라
매개변수화 타입은 불공변(invariant)이다.\
즉, 서로 다른 타입 Type1, Type2가 있을때  List\<Type1>은 List\<Type2>의 하위 타입도 상위타입도 아니다.\
List\<String>은 List\<Object>의 하위타입이 아니라는 뜻이다.\
List\<Object>에는 어떤 객체든 넣을수 있지만 List\<String>에는 문자열만 넣을 수 있다.\
List\<String>은 List\<Object>가 하는일을 제대로 수행하지 못하니 하위 타입이 될 수 없다. (리스코프 치환 원칙에 어긋난다.)

하지만 불공변방식보다 유연한 무언가가 필요하다.

Stack 클래스의 public API
```java
public class Stack<E> {
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
```
일련의 원소를 스택에 넣는 메서드를 추가
```java
public void pushAll(Iterator<E> src) {
  for (E e: src)
    push(e);
}
```

Iterable src의 원소 타입이 스택의 원소 타입과 일치하면 잘 작동한다.\
하지만 Stack\<Number>로 선언한 후 pushAll(intVal)을 호출하면 Integer는 Number의 하위타입이니 잘 동작할 것 같지만, 실제로 오류 메시지가 나타난다.\
매개변수화 타입이 불공변이기 때문이다.

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
numberStack.pushAll(integers);
```

```java
Error:(73, 29) java: incompatible types: java.lang.Iterable<java.lang.Integer> cannot be converted to java.lang.Iterable<java.lang.Number>
```
대처할 수 있는 한정적 와일드카드타입 이라는 특별한 매개변수화 타입을 지원한다.\
pushAll의 입력 매개변수 타입은 'E의 Iterable'이 아니라 'E의 하위 타입의 Iterable'이어야 한다.\
와일드카드 타입 Iterable<? extends E>가 정확히 이런뜻이다.
```java
public void pushAll(Iterable<? extends E> src) {
    for (E e: src) {
        push(e);
    }
}
```
Stack과 클라이언트 모두 컴파일되어, 타입 안전하다는 뜻이다.\
popAll 메서드 작성
```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```
```java
StackGeneric<Number> numberStack = new StackGeneric<>();
Collection<Object> objects = new ArrayList<>();
numberStack.popAll(objects);
```
```java
Error:(76, 28) java: incompatible types: java.util.Collection<java.lang.Object> cannot be converted to java.util.Collection<java.lang.Number>
```
Collection<Object>는 Collecton\<Number>의 하위 타입이 아니다 메시지가 나타난다.\
와일드카드타입으로 해결

popAll의 입력 매개변수의 타입이 'E의 Collecton'이 아니라 'E의 상위타입의 Collecton'이어야 한다.(모든타입은 자기 자신의 상위타입이다)\
Collecton<? super E>가 정확이 이런 뜻이다.

```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```
**_유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라_**

한편, 입력매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을게 없다.\
타입을 정확히 지정해야 하는 상황이므로, 이때는 와일드카드를 쓰지말아야 한다.

그럼 어떤 와일드카드 타입을 써야하는가???

펙스(PECS) : producer-extends, consumer-super

즉, 매개변수화 타입 T가 생성자라면 <? extends T>를 사용하고 소비자라면<? super T>를 사용

pushAll의 src 매개변수는 Stack이 사용할 E 인스턴스를 생산하므로 src의 적절한 타입은 Iterable<? extends E>이다.\
popAll의 dst 매개변수는 Stack으로부터 E 인스턴스를 소비하므로 dst의 적절한 타입은 Collection<? super E>이다.

Chooser 생성자
```java
public Chooser(Collection<T> choices)
```
이 생성자로 넘겨지는 choices컬렉션은 T타입의 값을 생산하기만 하니(그리고 나중을 위해 저장해 둔다.) T를 확장하는 와일드카드 타입을 사용해 선언해야 한다.

```java
Error:(25, 58) java: incompatible types: cannot infer type arguments for effect.item31.ChooserList<>
    reason: inference variable T has incompatible equality constraints java.lang.Number,java.lang.Integer
```

```java
public Chooser(Collection<? extends T> choices)
```

```java
public static void main(String[] args) {
    List<Integer> integers = Arrays.asList(1, 2, 3, 4);
    ChooserList<Number> chooserList = new ChooserList<>(integers);
    for (int i=0; i<5; i++) {
        System.out.println(chooserList.choose());
    }
}
```

unoin 메서드
```java
public static <E> Set<E> unoin(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```
s1, s2모두 생산자이므로 PESC 공식
```java
public static <E> Set<E> unoin(Set<? extends E> s1, Set<? extends E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```
```java
public void main(String[] args) {
    Set<Integer> integers = Set.of(1, 3, 5);
    Set<Double> doubles = Set.of(2.0, 5.0, 6.0);
    Set<Number> numbers = unoin(integers, doubles);
}
```
**클래스 사용자가 와일드카드 타입을 신경써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다.**

자바 8부터는 제대로 컴파일된다.\
자바 7까지는 타입 추론능력이 충분히 강력하지 못해서 문맥에 맞는 반환 타입(목표타입)을 명시해야 했다.

union 호출의 목표타입은 Set\<Number>이다.\
자바7까지는 Set.of 팩터리를 다른 것으로 적절히 변경한 후 이코드를 컴파일 하면 오류 메시지가 나타난다.\
컴파일러가 올바른 타입을 추론하지 못할때면 언제든지 명시적 타입 인수를 사용해서 타입을 알려주면된다.

```java
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

max 메서드
```java
public static <E extends Comparable<E>> E max(List<E> list)
```
입력 매개변수의 타입을 Collecton에서 List로 변경했다.

와일드카드 타입을 사용
```java
 public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

입력매개변수에서는 E 인스턴스를 생산하므로 List<? extends E> list
타입 매개변수 E가 Comparable<E>를 확장한다고 정의했는데, Comparable<E>는 E 인스턴스를 소비한다(그리고 선후 관계를 뜻하는 정수를 생산한다).
그래서 매개변수화 타입 Comparable<E>를 한정적 와일드카드 타입인 Comparable<? super E>로 대체했다.
Comparable은 언제나 소비자이므로, 일반적으로 Comparable<E>보다는 Comparable<? super E>를 사용하는 편이 낫다.
Comparator도 마찬가지다.
Comparator<E>보다는 Comparator<? super E>를 사용하는 편이 낫다.

```java
List<ScheduledFuture<?>> scheduledFutures = new ArrayList<>();
```

수정전 max가 이 리스트를 처리할 수 없는 이유?
(java.util.concurrent 패키지의)ScheduledFuture가 Comparable<ScheduledFuture>를 구현하지 않았기 때문이다. ScheduledFuture는 Delayed의 하위 인터페이스이고, Delayed는 Comparable<Delayed>를 확장했다.
scheduledFutures의 인스턴스는 다른 ScheduledFuture 인스턴스뿐만 아니라 Delayed 인스턴스와 비교할 수 있어서 수정전 max가 이 리스트를 거부하는 것이다.
더 일반화해서 말하면 Comparable(Comparator)을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해서 와일드 카드가 필요하다.

타임 매개변수와 와일드카드에는 공통되는 부분

```java
//비한정적 타입 매개변수
public static <E> void swap(List<E> list, int i, int j)
//비한정적 와일드카드
public static void swap(List<?> list, int i, int j)
```
public API라면 두번째가 더 낫다.
어떤 리스트든 이 메서드를 넘기면 명시한 인덱스의 원소들을 교환해 줄 것이다.
신경써야할 타입 매개변수도 없다.

기본규칙

**메서드 선언에 타입 매개변수가 한번만 나오면 와일드카드로 대체하라.**

이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드 카드로 바꾸면된다.

두번째 swap 문제
```java
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```
```java
Error:(36, 41) java: incompatible types: java.lang.Object cannot be converted to capture#1 of ?
```
방금 꺼낸 원소를 리스트에 다시 넣을 수 없는 오류로 원인은 리스트의 타입이 List<?>인데, List<?>에는 null외에는 어떤 값도 넣을 수 없다.\
(런타입 오류늘 낼 가능성이 있는) 형변환이나 리스트의 로 타입을 사용하지 않고도 해결할 길이 있다.\
바로 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용하는 방법이다.

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```
swapHelper 메서드는 리스트가 List/<E>임을 알고 있다.\
즉, 이 리스트에서 꺼낸 값의 타입은 항상 E이고, E타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다.\
swap 메서드 내부에서 더 복잡한 제네릭 메서드를 이용했지만, 외부에는 와일드 카드 기반의 선언을 유지할 수 있다.\
즉, swap 메서드를 호출하는 클라이언트는 복잡한 swapHelper의 존재를 모른 채 쓸 수 있다.

도우미 메서드의 시그니처는 앞에서 pulbic API로 쓰기엔 너무 복잡하단 이유로 제외했던 첫번재 swap 메서드의 시그니처와 완전히 똑같다.


_핵심정리_
> - 조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다.
> - 널리 쓰이는 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다.
> - PECS공식을 기억하자
> - 생산자(producer)는 extends를 소비자(consumer)는 super를 사용한다.
> - Comparable, Comparator는 모두 소비자이다.


[참조] (이펙티브 자바 3판)
