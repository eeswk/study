#### 상속을 고려해서 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

메서드를 재정의하면 어떤일이 일어나는지를 정확히 정리하여 문서로 남겨야 한다.
즉, 상속용 클래스는 재정의할 수 있는 메서드들은 내부적으로 어떻게 사용하는지(자기사용) 문서로 남겨야 한다.

재정의의 가능한 메서드
- public과 protected 메서드중 final이 아닌 모든 메서드

API문서의 메서드 설명 @implSpec 태그 사용

자바8에서 처음 도입되어 자바9에서 본격적으로 사용.
자바11의 자바독에서는 선택사항
~~~java
 -tag "implSpec:a:implementation Requirements:"를 지정
~~~

자바8
~~~~java
public abstract class AbstractCollection<E> implements Collection<E> {
  /**
   * {@inheritDoc}
   *
   * <p>This implementation iterates over the collection looking for the
   * specified element.  If it finds the element, it removes the element
   * from the collection using the iterator's remove method.
   *
   * <p>Note that this implementation throws an
   * <tt>UnsupportedOperationException</tt> if the iterator returned by this
   * collection's iterator method does not implement the <tt>remove</tt>
   * method and this collection contains the specified object.
   *
   * @throws UnsupportedOperationException {@inheritDoc}
   * @throws ClassCastException            {@inheritDoc}
   * @throws NullPointerException          {@inheritDoc}
   */
  public boolean remove(Object o) {
      Iterator<E> it = iterator();
      if (o==null) {
          while (it.hasNext()) {
              if (it.next()==null) {
                  it.remove();
                  return true;
              }
          }
      } else {
          while (it.hasNext()) {
              if (o.equals(it.next())) {
                  it.remove();
                  return true;
              }
          }
      }
      return false;
  }
}
~~~~
자바11 @implSpec
~~~java
public abstract class AbstractCollection<E> implements Collection<E> {
  /**
   * {@inheritDoc}
   *
   * @implSpec
   * This implementation iterates over the collection looking for the
   * specified element.  If it finds the element, it removes the element
   * from the collection using the iterator's remove method.
   *
   * <p>Note that this implementation throws an
   * {@code UnsupportedOperationException} if the iterator returned by this
   * collection's iterator method does not implement the {@code remove}
   * method and this collection contains the specified object.
   *
   * @throws UnsupportedOperationException {@inheritDoc}
   * @throws ClassCastException            {@inheritDoc}
   * @throws NullPointerException          {@inheritDoc}
   */
  public boolean remove(Object o) {
      Iterator<E> it = iterator();
      if (o==null) {
          while (it.hasNext()) {
              if (it.next()==null) {
                  it.remove();
                  return true;
              }
          }
      } else {
          while (it.hasNext()) {
              if (o.equals(it.next())) {
                  it.remove();
                  return true;
              }
          }
      }
      return false;
  }
}
~~~
클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야할 수도 있다.
~~~~java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {

/**
     * Removes from this list all of the elements whose index is between
     * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
     * Shifts any succeeding elements to the left (reduces their index).
     * This call shortens the list by {@code (toIndex - fromIndex)} elements.
     * (If {@code toIndex==fromIndex}, this operation has no effect.)
     *
     * <p>This method is called by the {@code clear} operation on this list
     * and its subLists.  Overriding this method to take advantage of
     * the internals of the list implementation can <i>substantially</i>
     * improve the performance of the {@code clear} operation on this list
     * and its subLists.
     *
     * @implSpec
     * This implementation gets a list iterator positioned before
     * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
     * followed by {@code ListIterator.remove} until the entire range has
     * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
     * time, this implementation requires quadratic time.</b>
     *
     * @param fromIndex index of first element to be removed
     * @param toIndex index after last element to be removed
     */
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }
}
~~~~

removeRange 메서드는 하위 클래스에서 부분 리스트의 clear 메서드를 고성능으로 만들기 쉽게 하기 위해 만들었다.
하위클래스에서 clear 메서드를 호출하면 (제거할 원소 수의) 제곱 비례해 성능이 느려지거나 부분리스트의 매커니즘을 밑바닥부터 새로 구현해야 했을 것이다.

상속용 클래스에 protected 사용 범위?
> 실제 하위 클래스에서 만들어서 시험해보는 것이 최선.
> 상속클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다.

하위클래스를 여러개 만들때 까지 전혀 쓰이지 않는 protected 멤버는 사실 private이었야할 가능성이 크다.
하위클래스 3개정도가 적당하고, 이 중 하나 이상은 제 3자가 작성해봐야 한다.

즉, 널리 쓰일 클래스를 상속용으로 설계한다면 문서화한 내용 사용패턴과 protected 메서드와 필드를 구현하면서 선택 결정에 영원히 책임져야 함을 인식해야 한다.
상속용으로 설계한 클래스는 배포전에 반드시 하위 클래스를 만들어 검증해야 한다.

상속용 클래스 제약사항
1. 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.
~~~java
package effect;

public class Item19 {

    public Item19() {
        overrideMe();
        overrideNotMe();
        overrideNotStaticMe();
        overrideNotFinalMe();
    }
    //상속가능
    public void overrideMe() { }
    //상속가능
    protected void overrideMe1() { }
    //상속가능
    void overrideMe2() {}


    //상속불가
    private void overrideNotMe() { }
    //상속불가
    public static void overrideNotStaticMe() { }
    //상속불가
    public final void overrideNotFinalMe() { }
}

package effect;

import java.time.Instant;

public class Item19Sub extends Item19{
    //초기화되지 않는 final 필드, 생성자에서 초기화한다.
    private final Instant instant;

    public Item19Sub() {
        instant = Instant.now();
    }
    //재정의 가능 메서드. 상위 클래스의 생성자가 호출된다.
    @Override
    public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Item19Sub sub = new Item19Sub();
        sub.overrideMe();
    }

}

~~~
결과
~~~java
null
2019-01-15T01:00:43.008Z
~~~
설명
~~~java
상위클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출한다.

해결방법
private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.
~~~
2. clone과 readObejct 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.

**클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스 안기는 제약도 상당하다.**
> #### 해결방법
> - 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것.
> 1. 클래스를 final로 선언
> 2. private나 package-private으로 선언하고 public 정적 팩터리를 만든다.

꼭 상속을 허용해야한다면, 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이 사실을 문서로 남기는 것이다.
재정의 가능 메서드를 호출하는 자기 코드를 완벽히 제거하는 것.
메서드를 재정의해도 다른 메서드의 동작에 아무런 영향을 주지 않기 때문이다.

방법?
각각의 재정의 가능 메서드는 자신의 본문 코드를 private '도우미 메서드' 옮기고, 이 메서드를 호출하도록 수정한다. 그런 다음 재정의 가능 메서드를 호출하는 다른 코드들도 모두 도무이 메서드를 직접 호출하도록 수정하면된다.

_핵심정리_

> - 상속용 클래스를 설계하기란 결코 만만치 않다.
> - 클래스 내부에서 스스로를 어떻게 사용하는지(자기사용 패턴) 모두 문서로 남긴다.
> - 문서화한 것은 그 클래스가 쓰이는 한 반드시 지켜야 한다.
> - 그렇지 않으면 그 내부 구현방식을 믿고 활용하던 하위 클래스를 오작동하게 만들수 있다.
> - 다른 이가 효율 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected로 제공해야 할 수도 있다.
> - 클래스를 확장해야 할 명확한 이유가 떠오르지 않으면 상속을 금지하는 편이 낫다.
> - 상속을 금지하려면 클래스를 final로 선언하거나 생성자 모두를 외부에서 접근할 수 없도로고 만들면 된다.

[참조] (이펙티브 자바 3판)
