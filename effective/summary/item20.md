#### 추상클래스보다는 인터페이스를 우선하라

자바가 제공하는 다중 구현 메커니즘
- 인터페이스
- 추상클래스

차이점?
추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.

인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규칙을 잘 지킨 클래스라면 어떤 클래스를 상속했든 같은 타입으로 취급

인터페이스
- 믹스인(mixin)정의에 안성맞춤이다.(예 Comparable)
- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들수있다.
~~~java
package effect.item20;

import effect.item20.Song;

import java.applet.AudioClip;

public interface Singer {
    AudioClip sing(Song s);
}
~~~
~~~java
package effect.item20;

public interface Songwriter {
    Song compose(int chartPostion);
}
~~~
~~~java
package effect.item20;

import java.applet.AudioClip;

public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}
~~~

인터페이스의 메서드중 구현방법이 명백한 것이 있다면 그 구현을 디폴트 메서드로 제공해 프로그래머들의 일감을 덜어줄 수 있다.

~~~java
public interface Collection<E> extends Iterable<E> {


  /**
   * Removes all of the elements of this collection that satisfy the given
   * predicate.  Errors or runtime exceptions thrown during iteration or by
   * the predicate are relayed to the caller.
   *
   * @implSpec
   * The default implementation traverses all elements of the collection using
   * its {@link #iterator}.  Each matching element is removed using
   * {@link Iterator#remove()}.  If the collection's iterator does not
   * support removal then an {@code UnsupportedOperationException} will be
   * thrown on the first matching element.
   *
   * @param filter a predicate which returns {@code true} for elements to be
   *        removed
   * @return {@code true} if any elements were removed
   * @throws NullPointerException if the specified filter is null
   * @throws UnsupportedOperationException if elements cannot be removed
   *         from this collection.  Implementations may throw this exception if a
   *         matching element cannot be removed or if, in general, removal is not
   *         supported.
   * @since 1.8
   */
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
}
~~~
주의)
많은 인터페이스가 equals와 hashCode 같은 Object의 메서드를 정의하고 있지만 디폴드 메서드로 제공해서는 안된다.

인터페이스와 추상골격구현(skeletal implementation)클래스를 함께 제공하는 식으로 인터페이스와 추상클래스의 장점을 모두 취하는 방법

인터페이스로는 타입을 정의하고 필요하면 디폴트 메서드 몇개도 함께 제공한다.
그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다. (템플릿 메서드 패턴)

인터페이스 이름이 Interface라면 그 골격 구현 클래스의 이름은 AbstractInterface
예) 컬렉션 프레임워크의 AbstractCollection, AbstractSet, AbstractList, AbstractMap 핵심 컬렉션 인터페이스의 골격 구현이다.

골격구현은(독립된 추상 클래스나 디폴드 메서드를 사용하는 인터페이스) 그 인터페이스로 나름 구현을 만들려는 프로그래머의 일을 상당히 덜어준다.

예) AbstractList 골격구현으로 활용
~~~java
package effect.item20;

import java.util.AbstractList;
import java.util.Arrays;
import java.util.List;
import java.util.Objects;

public class CustomerList {
    //정적 팩터리 메서드
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);
        //다이아몬드 연산자를 이렇게 사용하는건 자바 9부터 가능하다.
        //더 낮은 버전은 사용한다면 <Integer>로 수정
        return new AbstractList<Integer>() {
            @Override
            public Integer get(int index) {
                return a[index];    //오토박싱
            }

            @Override
            public Integer set(int index, Integer element) {
                int oldVal = a[index];
                a[index] = element; //오토언박싱
                return oldVal;  //오토박싱
            }

            @Override
            public int size() {
                return a.length;
            }
        };
    }

    public static void main(String[] args) {
        int [] ints = {4,5,6};
        List testList = CustomerList.intArrayAsList(ints);
        System.out.println(testList.toString());
        testList.set(2,7);
        System.out.println(testList.toString());
    }
}
~~~
결과
~~~java
[4, 5, 6]
[4, 5, 7]
~~~
문제점
- int값과 Integer 인스턴스 사이의 변환(박싱, 언박싱) 때문에 성능은 좋지 않음
- 익명클래스형태 사용

골격 구현 클래스의 좋은점은 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할때 따라오는 제약에서는 자유롭다는 점.
골격 구현을 확장하는 것으로 인터페이스 구현이 거의 끝나지만, 꼭 이렇게 해야 하는 것은 아니다. 구조상 골격 구현을 확장하지 못하는 처지라면 인터페이스를 직접 구현해야 한다. (디폴트 메서드)
골격 구현 클래스를 우회적으로 이용할 수 있다.
인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다.

골격구현작성
1. 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드를 선정한다. (이 기반 메서드들은 골격 구현에서는 추상 메서드가 될 것이다.)
2. 기반 메서드를 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공한다. (equals, hashCode같은 Object의 메서드는 디폴트 메서드 제공 X, 인터페이스 메서드 모두가 기반 메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 별도로 만들 이유가 없다.)
3. 기반 메서드나 디폴트 메서드가 만들지 못한 메서드가 남았다면, 이 인터페이스를 구현하는 골격 클래스를 하나 만들어 남은 메서드들을 작성해 넣는다. (골격 구현 클래스에는 필요하면 public이 아닌 필드와 메서드를 추가해도 된다.)

Map.Entry 인터페이스에 getKey, getValue는 기반 메서드, 선택적으로 setValue도 할 수 있다.
equals와 hashCode의 동작방식도 정의했다.
Object 메서드들은 디폴트 메서드로 제공해서는 안되므로 해당 메서드는 모두 골격 구현 클래스에 구현한다. toString도 기반 메서드를 사용해 구현해놨다.

예) 골격구현 클래스
~~~java
package effect.item20;

import java.util.Map;
import java.util.Objects;

public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V>{

    //변경가능한 엔트리는 이 메서드를 반드시 재정의해야한다.
    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    //Map.Entry.equals의 일반 규약을 구현
    @Override
    public boolean equals(Object obj) {
        if (obj == this)
            return true;
        if (!(obj instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) obj;
        return Objects.equals(e.getKey(), getKey())
                && Objects.equals(e.getValue(), getValue());
    }
    //Map.Entry.hashCode의 일반 규약을 구현
    @Override
    public int hashCode() {
        return Objects.hashCode(getKey())
                ^ Objects.hashCode(getValue());
    }

    @Override
    public String toString() {
        return getKey() + "=" + getValue();
    }
}
~~~
Map.Entry 인터페이스나  그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다. 디폴트 메서드는 equals, hashCode, toString 같은 Object 메서드를 재정의 할 수 없기 때문이다.

골격구현은 기본적으로 상속해서 사용하는 걸 가정하므로 설계 및 문서화 지침을 모두 따라야 한다.
단순 구현(simple implementation)은 골격 구현의 작은 변종으로 AbstractMap.SimpleEntry가 좋은 예이다.

단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니라는 점이 다르다. (동작하는 가장 단순한 구현)
단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 된다.

_핵심정리_

> - 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다.
> - 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 고려해보자.
> - 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다.
> - '가능한 한'이유는, 인터페이스에 걸려있는 구현상의 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.

[참조] (이펙티브 자바 3판)
