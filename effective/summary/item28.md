#### 배열보다는 리스트를 사용하라
배열과 제네릭 타입에는 중요한 차이점

> 첫번재

배열은 공변(convariant)이다.\
Sub가 Super의 하위타입이라면 배열Sub[]은 배열 Super[]의 하위 타입이 된다.(공변, 함께 변한다는 뜻이다.)

제네릭은 불공변(invariant)이다.\
서로 다른 타입 type1과 type2가 있을때, List\<type1>은 List\<type2>의 하위 타입도 아니고 상위 타입도 아니다.

```java
public class Item28 {
    public static void main(String[] args) {
        Object[] objectArray = new Long[1];
        objectArray[0] = "타입이 달라 넣을 수 없다.";
    }
}
```
결과 런타임오류
```java
Exception in thread "main" java.lang.ArrayStoreException: java.lang.String
	at effect.item28.Item28.main(Item28.java:14)
```
```java
public class Item28 {
    public static void main(String[] args) {
      List<Object> ol = new ArrayList<Long>();
      ol.add("타입이 달라 넣을 수 없다.");
    }
}
```
결과 컴파일오류
```java
Error:(13, 27) java: incompatible types: java.util.ArrayList<java.lang.Long>
cannot be converted to java.util.List<java.lang.Object>
```
어느쪽으든 Long용 저장소에 String을 넣을 수 없지만, 배열에서는 그 실수를 런타임에서 알게 되고, 리스트를 사용하면 컴파일할 때 바로 알 수 있다.

> 두번째

배열은 실제화(reify)된다. \
즉, 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
Long배열에 String을 넣으려하면 ArrayStoreException이 발생한다.

제네릭은 타입 정보가 런타임에는 소거(erasure)된다. \
소거는 제네릭이 지원되기 전에 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 해주는 매커니즘으로, 자바5가 제네릭으로 전환될 수 있도록 해줬다.

따라서 배열과 제네릭은 잘 어우러지지 못한다.

배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.\
코드를 new List\<E> [], new List\<String> [], new E []식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킨다.

제네릭 배열을 만들지 못하는 이유?\
타입 안전하지 않기 때문이다.\
이를 허용하면 컴파일러가 자동으로 생성한 형변환 코드에서 런타임에 ClassCastException이 발생할 수 있다. \
런타임에 ClassCastException이 발생하는 일을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋나는 것이다.

```java
List<String> [] stringLists = new List<String>[1];  //1
List<Integer> intList = List.of(42);                //2
Object[] objects = stringLists;                     //3
objects[0] = intList;                               //4
String s = stringLists[0].get(0);                   //5
```
만약 제네릭 배열을 생성을 허용하게 된다면,\
(1)이 가능하다면,\
(2)는 원소가 하나인 List\<Integer>를 생성한다.\
(3)은 (1)에서 생성한 List\<String>의 배열을 Object 배열에 할당한다.
(배열은 공변이니 문제없음)\
(4)는 (2)에서 생성한 List\<Integer>의 인스턴스를 Object 배열의 첫 원소로 저장한다.(제네릭은 소거방식으로 구현되어 문제없다.)\
즉, 런타임에는 List\<Integer> 인스턴스의 타입은 단순히 List가 되고, List\<Integer>[] 인스턴스의 타입은 List[]가 된다.\
ArrayStoreException 발생하지 않는다.\
(5)List\<String> 인스턴스만 담겠다고 선언한 stringLists 배열에는 지금 List\<Integer> 인스턴스가 저장돼 있다.\
그리고 이 배열의 처음 리스트에서 첫 원소를 꺼내려 한다.\
컴파일러는 꺼낸 원소를 자동으로 String으로 형변환하는데, 이 원소를 Integer이므로 런타임에 ClassCastException이 발생한다.\
즉 이런 일을 방지하려면 (제네릭 배열이 생성되지 않도록) (1)에서 컴파일 오류를 내야 한다.

E, List\<E>, List\<String>같은 타입을 실체화 불가 타입(non-reifiable type) 이라고 한다.\
실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.\
소거 매커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 List<?>와 Map<?, ?> 같은 비한정적 와일드카드타입뿐이다.\
배열을 비한정적 와일드카드 타입으로 만들 수는 있지만 유용하게 쓰일 일은 거의 없다.

배열을 제네릭으로 만들 수 없어 귀찮을 때도 있다.\
제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는게 보통은 불가능하다.\
또한 제네릭 타입과 가변인수(vararge method)를 함께 쓰면 해석하기 어려운 경고 메시지를 받게 된다.\
가변인수 메서드를 호출 할 때마다 가변인수 매개변수를 담은 배열이 하나 만들어지는데, 이때 그 배열의 원소가 실체화 불가 타입이라면 경고가 발생하는 것이다.\
이 문제는 @SafeVarargs 애너테이션으로 대처할 수 있다.

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우는 대부분은 배열인 E[] 대신 컬렉션 List\<E>를 사용하면 해결된다. \
코드가 조금 복잡해지고 성능이 나빠질 수 있지만, 그 대신 타입 안정성과 상호운용성은 좋아진다.

생성자에서 컬렉션을 받는 Chooser 클래스\
이 클래스는 컬렉션 안의 원소 중 하나를 무작위로 선택해 반환하는 chooose 메서드를 제공한다.\
생성자에 어떤 컬렉션을 넘기느냐에 따라 이 클래스를 주사위판, 매직 8볼, 몬테카를로 시뮬레이션용 데이터 소스 등으로 사용할 수 있다.

```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }
    public Object choose() {
        Random random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)];
    }

    public static void main(String[] args) {
        Chooser chooser = new Chooser(Arrays.asList("1","2","3","4","5","6"));
        for (int i =0; i<chooser.choiceArray.length; i++) {
            System.out.println(chooser.choose());
        }
    }
}
```
choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다.
타입이 다른 원소가 들어 있었다면 런타임에 형병환 오류가 발생한다.

제네릭으로 수정
```java
public class ChooserGeneric<T> {
    private final T[] choiceArray;

    public ChooserGeneric(Collection<T> choices) {
        this.choiceArray = choices.toArray();
    }
    public Object choose() {
        Random random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)];
    }
}
```
컴파일 오류
```java
Error:(12, 43) java: incompatible types: java.lang.Object[] cannot be converted to T[]
```
Object 배열을 T 배열로 형변환 수정
```java
 choiceArray = (T[]) choices.toArray();
```
경고 발생
```java
Warning:(12, 44) java: unchecked cast
  required: T[]
  found:    java.lang.Object[]
```
T가 무슨 타입인지 알수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다.\
제네릭에서는 원소의 타입 정보가 소거되어 런타임에는 무슨 타입인지 알 수 없을 기억하자.\
프로그램은 동작하지만 컴파일러가 안전을 보장하지 못할 뿐이다.\
코드를 작성하는 사람이 안전하다고 확신한다면 주석을 남기고 애너테이션을 달아 경고를 숨겨도 된다.\
하지만 애초에 경고의 원인을 제거하는 편이 훨씬 낫다.

비검사 형변환 경고를 제거하라면 배열 대신 리스트를 쓰면된다.

```java
public class ChooserList<T> {
    private final List<T> choiceList;

    public ChooserList(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }
    public Object choose() {
        Random random = ThreadLocalRandom.current();
        return choiceList.get(random.nextInt(choiceList.size()));
    }
}
```
런타임에 ClassCastException 발생하지 않는다.

_핵심정리_
> - 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다.
> - 배열은 공변이고, 실체화된다.
> - 제네릭은 불공변이고, 타입정보가 소거된다.
> - 배열은 런타임에는 안전하지만, 컴파일타임에는 안전하지 못하다.
> - 제네릭은 컴파일타임에는 안전하지만, 런타임에는 안전하지 못하다.
> - 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장먼저 배열을 리스트로 대체하는 방법을 적용하자.






[참조] (이펙티브 자바 3판)
