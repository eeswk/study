#### 제네릭과 가변 인수를 함께 쓸때 신중하라.
가변인수(Varargs) 메서드와 제네릭은 자바 5때 함게 추가되었다.

가변인수를 이용한 메서드
```java
public class VarargsMain {
  public static void varargsTest(String... strs) {
    for(String s : strs)
      System.out.println("가변인수 형태: " + s);
  }
}

VarargsMain.varargsTest("a", "b", "c");

```
가변 인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할수 있게 해주는데, 구현 방식에 허점이 있다\
가변인수 메서드르 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다.\
그런데 내부로 감춰야 했을 이 배열을 그만 클라이언트에 노출하는 문제가 생겼다.\
그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입을 포함되면 알기 어려운 컴파일 경고가 발생한다.

제네릭과 varargs를 혼용하면 타입 안정성이 깨진다!
```java
static void dangerous(List<String>...stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; //힙오염
    String s = stringLists[0].get(0) //ClassCastException
}
```


_핵심정리_
> -


[참조] (이펙티브 자바 3판)
