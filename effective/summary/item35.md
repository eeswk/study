#### ordinal 메서드 새신 인스턴스 필드를 사용하라.
열거타입 상수는 자연스럽게 하나의 정수값에 대등된다.
해당 상수가 그 열거 타입에서 몇번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다.
```java
public enum Ensemble {
  FIRST, SECOND, THIRD, FOURTH, FIFTH, TEN;

  public int numberOfOrdinals() { return ordinal() + 1 ; }
}
```
문제점
- 상수 선언 순서를 바꾸는 순간 numberOfOrdinals가 오동작하며, 이미 사용중인 정수와 같은 상수는 추가할 방법이 없다.
- 값을 중간에 비워둘수도 없다.

해결
- 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장한다.

 ```java
 public enum Ensemble {
   FIRST(1), SECOND(2), THIRD(3), FOURTH(4), FIFTH(5), TEN(10);

   private final int numberOfOrdinals;
   Ensemble (int Size) { this.numberOfOrdinals = size; }
   public int numberOfOrdinals() { return numberOfOrdinals; }
 }
 ```


_핵심정리_
> - 이 메서드는 EnumSet 과 EnumMap과 같이 열거 타입기반의 범용 자료 구조에 쓸 목적으로 설계되었다.
> - ordinal 메서드는 절대 사용하지 말자.


[참조] (이펙티브 자바 3판)
