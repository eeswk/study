#### 상속보다는 컴포지션을 사용하라

상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다.
잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다.

일반적인 구체 클래스를 패키지 경계를 넘어, 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.
상속은 클래스가 다른 클래스를 확장하는 구현 상속을 말한다.
클래스가 인터페이스를 구현하거나 인터페이스가 다른 인터페이스를 확장하는 인터페이스 상속과는 무관하다.

메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.
상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 건드리지 않는 하위 클래스가 오동작할 수 있다는 것이다.
이러한 이유로 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 발맞춰 수정해야만 한다.

~~~java
package effect;

import java.util.Arrays;
import java.util.Collection;
import java.util.HashSet;

public class InstrumentedHashSet<E> extends HashSet<E> {

    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
    }
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(Arrays.asList("A","B","C"));
        System.out.println(s.getAddCount());
    }
}
~~~

_설명_

인스턴스 addAll 메서드로 원소 3개를 더했다고 가정하면,
정적팩토리 메서드인 List.of로 리스트를 생성 또는 Arrays.asList로 생성하면 getAddCount의 반환값은 6이다.

원인은 HashSet의 addAll 메서드가 add 메서드를 사용해 구현된데 있다.
이런 내부 구현 방식은 HashSet 문서에는 쓰여있지 않다. InstrumentedHashSet의 addAll은 addCount에 3을 더한 후 HashSet의 addAll 구현을 호출했다.
HashSet의 addAll은 각 원소를 add 메서드를 호출해 추가하는데, 이때 add는 InstrumentedHashSet에서 재정의한 메서드다.
따라서 addCount에 값이 중복해서 더해져, 최종값이 6으로 늘어난 것이다.
addAll로 추가한 원소 하나당 2씩 늘어났다.
하위 클래스에서 addAll 메서드를 재정의하지 않으면 문제를 고칠수 있다.
하지만 HashSet의 addAll이 add메서드를 이용해 구현했음을 가정한 해법이라는 한계를 지닌다.

상위 클래스에 새로운 메서드가 추가될 경우 하위 클래스가 깨지기 쉽다.

**_해결 방법?_**

기존 클래스를 확장하는 대신 새로운 클래스르 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.
기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션(composition 구성)이라고 한다.
새 클래스의 인스턴스 메서들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.
이 방식을 전달방식(forwarding method)라 부른다.
새로운 클래스는 기존 클래스의 내부 구현 방식의 영향을 벗어나며, 기존 클래스에 새로운 메서드를 추가되더라도 영향을 받지 않는다.

~~~java
//래퍼 클래스 - 상속대신 컴포지션을 사용
package effect;

import java.time.Instant;
import java.util.*;

public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
~~~
~~~java
//재사용할 수 있는 전달 클래스
package effect;

import java.util.Collection;
import java.util.Iterator;
import java.util.Set;

public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    @Override
    public int size() {
        return s.size();
    }

    @Override
    public boolean isEmpty() {
        return s.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return s.contains(o);
    }

    @Override
    public Iterator<E> iterator() {
        return s.iterator();
    }

    @Override
    public Object[] toArray() {
        return s.toArray();
    }

    @Override
    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean add(E e) {
        return s.add(e);
    }

    @Override
    public boolean remove(Object o) {
        return s.remove(o);
    }

    @Override
    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    @Override
    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    @Override
    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    @Override
    public void clear() {
        s.clear();
    }

    @Override
    public boolean equals(Object o) {
        return s.equals(o);
    }

    @Override
    public int hashCode() {
       return s.hashCode();
    }

    @Override
    public String toString() {
        return  s.toString();
    }
}
~~~

**_설명_**

InstrumentedSet은 HashSet의 모 든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다.
구체적으로는 Set 인터페이스를 구현했고 Set의 인스턴스를 인수받는 생성자를 하나 제공한다. 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다.
상속방식은 구체 클래스 각각 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다.
컴포지션 방식은 한번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며 기존 생성자들과 함께 사용할 수 있다.

InstrumentedSet을 이용하면 대상 Set 인스턴스를 특정 조건하에서만 임시로 계측할 수 있다.

다른 Set 인스턴스를 감싸고(wrap) 있다는 뜻에서 InstrumentedSet 같은 클래스를 래퍼 클래스라고 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다.
컴포지션과 전달의 조합은 넓은 의미로 위임(delegation)이라고 부른다.
엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신을 참조를 넘기는 경우만 위임에 해당한다.

래퍼 클래스는 단점이 거의 없다.
단 래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다는 점만 주의하면된다.
콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다른 호출(콜백) 때 사용하도록 한다.
내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백때는 래퍼가 아닌 내부 객체를 호출하게 된다.
이를 SELF문제라고 한다.
전달 메서드가 성능에 주는 영향이나 래퍼 객체가 메모리 사용량에 주는 영향을 걱정할 수 있지만, 실전에서는 둘다 별다른 영향이 없다고 밝혀졌다.

전달 메서드를 작성하는게 지루하겠지만, 재사용할 수 있는 전달 클래스를 인터페이스당 하나씩만 만들어두면 원하는 기능을 덧씌우는 전달 클래스들을 아주 손쉽게 구현할 수있다.
구아바는 모든 컬렉션 인터페이스용 전달 메서드를 전부 구현해뒀다.

_결론_

상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다.
클래스 B가 클래스A와 is-a 관계일때만 클래스 A를 상속해야 한다.
클래스 A를 상속하는 클래스 B를 작성하려면 "B가 정말 A인가?" 확인하고 아니라면 B는 A를 상속하면 안된다.
A를 private 인스턴스로 두고 A와 다른 API를 제공해야하는 상황이 대다수다.
즉 A는 B의 필수 구성요소가 아니라 구현하는 방법 중 하나일 뿐이다.

컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하게 된다. 그 결과 API가 내부 구현에 묶이고 그 클래스의 성능도 제한된다.
클라이언트가 노출된 내부에 직접 접근할 수 있다는 점이다.
사용자를 혼란스럽게 할 수 있다.

컴포지션 대신 상속을 사용하기로 결정하기 전에 마지막으로 자문해야 할 질문?
확장하려는 클래스의 API에 아무런 결함이 없는가?
결함이 있다면 클래스의 API까지 전파되도 괜찮은가?
컴포지션으로 이런 결함을 숨기는 새로운 API를 설계할 수 있지만, 상속은 상위 클래스의 API를 그 결함까지도 그대로 승계한다.

_핵심정리_

상속은 강력하지만 캡슐화를 해치는 문제가 있다.
상속은 상위클래스와 하위 클래스가 순수한 is-a 관계일때만 써야한다.
is-a 관계일때도 하위클래스의 패키지가 상위클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 문제가 될수 있다.
상속의 취약점을 피하려면 상속대신 컴포지션과 전달을 사용하자.
특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 사용하자.
래퍼 클래스는 하위 클래스보다 견고하고 강력하다.
