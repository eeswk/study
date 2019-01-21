#### 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가할 방법이 없었다.
인터페이스에 메서드를 추가하면 컴파일 오류.
추가된 메서드가 기존 구현체에 이미 존재할 가능성은 낮다.
자바 8부터 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드가 생겼다.

디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의 하지 않은 모든 클래스에 디폴트 구현이 쓰이게 된다.
즉, 기존 인터페이스에 메서드를 추가할 수 있지만, 모든 구현체들과 매끄럽게 연동된다는 보장은 없다.

자바7까지는 모든클래스가 '현재의 인터페이스에 새로운 메서드가 추가될 일은 없다고'가정했다.

자바8에서는 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가되었다.
주로 람다를 활용하기 위해서다.
자바 라이브러리의 디폴트 메서드는 코드 품질을 높고 범용적이라 대부분 상황에서 잘 동작한다.
**하지만 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다.**

자바 8 Collection 인터페이스에 추가된 removeIf 메서드
```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```
불리언함수(predicate)가 true를 반환하는 모든 원소를 제거한다.
디폴트 구현은 반복자를 이용해 순회하면서 각 원소를 인수로 넣어 프레디키트를 호출하고, 프레디키트가 true를 반환하면 반복자의 remove 메서드를 호출해 그 원소를 제거한다.

org.apache.commons.collections4.collection.SynchronizedCollection
아파치 커먼즈 라이브러리의 이 클래스는 java.util의 Collections.synchronizedCollection 정적 팩터리 메서드가 반환하는 클래스와 비슷하다.

_문제점_
아파치 버전은 (컬렉션 대신) 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다. 즉, 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스이다.

아파치의 SynchronizedCollection 클래스는 removeIf 메서드를 재정의하지 않고 있다. 이 클래스를 자바 8과 함께 사용한다면(removeIf의 디폴트 구현을 물려받게 된다면), 모든 메서드 호출을 알아서 동기화 해주지 못한다.
removeIf의 구현은 동기화에 관해 아무것도 모르므로 락 객체를 사용할 수 없다. 따라서 SynchronizedCollection 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가 removeIf를 호출하면 ConcurrentModificationException이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.

_예방법_
자바 플랫폼 라이브러리에서는 구현한 인터페이스의 디폴트 메서드를 재정의하고, 다른 메서드에서 디폴트 메서드를 호출하기전에 필요한 작업을 수행하도록 했다.
Collections.synchronizedCollection이 반환하는 package-private 클래스들은 removeIf를 재정의하고, 이를 호출하는 다른 메서드들은 디폴트 구현을 호출하기 전에 동기화를 하도록 했다.
하지만 자바 플랫폼에 속하지 않는 제 3의 기존 컬렉션 구현체들은 이런 언어 차원의 인터페이스의 변화에 맞춰 수정될 기회가 없었다.

**디폴트 메서드는(컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있다.**
- 기존 인터페이스에 디폴트 메서드 추가는 꼭 필요한 경우아니면 피한다.
- 새로운 인터페이스는 표준적인 메서드 구현을 제공하는데 유용한 수단이 된다.

디폴트 메서드가 생겼더라도 **인터페이스를 설계 할 때는 여전히 세심한 주의를 기울여야 한다**

새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야 한다.
**인터페이스를 릴리스한 후라도 결함을 수정하는게 가능한 경우가 있겠지만, 절대 그 가능성에 기대서는 안된다.**


test 예제
```java
public class Item20 {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<Integer>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.removeIf(i -> i%2 == 0);
        System.out.println(list);

        List<Integer> list1 = Arrays.asList(1,2,3,4);
        list1.set(2, 5);
        System.out.println(list1);
        //사용불가
//        list1.removeIf(i -> i%2==0);

//        default void remove() {
//          throw new UnsupportedOperationException("remove");
//        }


        List<String> test = new ArrayList();
        test.add("A");
        test.add("B");
        test.add("C");
        test.add("D");
        System.out.println("test ArrayList before :" + test);
        test.removeIf(t -> t.equals("B"));
        //default removeIf use
        System.out.println("test ArrayList removeIf after :" + test);

        Collection<String> stringCollection = Collections.synchronizedCollection(test);
        System.out.println("Sunchronized view is :"+stringCollection);
        stringCollection.removeIf(t -> t.equals("A"));
        System.out.println("Sunchronized removeIf :"+stringCollection);
```

[참조] (이펙티브 자바 3판)
