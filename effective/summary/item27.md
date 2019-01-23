#### 비검사 경고를 제거하라.

제네릭 사용시 많은 컴파일러 경고가 나타난다.
> - 비검사 형변환 경고
> - 비검사 메서드 호출 경고
> - 비검사 매개변수화 가변수인 타입경고
>  - 비검사 변환경고
대부분의 비검사 경고는 쉽게 제거할 수 있다.

```java
public class Item27 {
    Set<Lark> exaltation = new HashSet();  //경고노출

    public static void main(String[] args) {
    }
}
```

javac 명령줄 인수에 -Xlint:unchecked 옵션추가
```
intellij case setting
settings -> compiler -> java Compiler
Addtional command line parameters:
-Xlint:unchecked
```
컴파일시 경고 노출
```java
Warning:(7, 28) java: unchecked conversion
  required: java.util.Set<effect.item27.Lark>
  found:    java.util.HashSet
```
컴파일러가 알려준 타입 매개변수를 명시하지 않고 다이아몬드 연산자(<>)만으로 해결할 수 있다.
```java
Set<Lark> exaltation = new HashSet<>();
```

**할수 있는 한 모든 비검사 경고를 제거하라** \
모두 제거한다면 그 코드는 타입 안전성이 보장된다. (런타임에 ClassCastException이 발생할 일이 없다.)

**경고를 제거할 수는 없지만 타입 안전하다고 확실할 수 있다면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨길 수 있다.**

@SuppressWarnings 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달수 있다. 하지만 @SuppressWarnings 애너테이션은 항상 가능한 좁은 범위에 적용하는 것이 좋다.

ArrayList
```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```
컴파일 결과
```java
Warning:(25, 39) java: unchecked cast
  required: T[]
  found:    java.lang.Object[]
```

애너테이션은 선언에만 달 수 있기 때문에 return 문에 @SuppressWarnings를 다는게 불가능하다.
메세드 전체에 달고 싶지만, 범위가 필요 이상으로 넓어진다.
따라서 반환값을 담을 지연변수를 하나 선언하고 그 변수를 애너테이션을 달아준다.

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        //생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 값으므로
        //올바른 형병환이다.
        @SuppressWarnings("unchecked")
        T[] result =
                (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```
**@SuppressWarnings 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.**

_핵심정리_
> - 비검사 경고는 중요하니 무시하지 말자.
 >- 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거하라.
> - 경고를 없앨방법을 찾지 못하겠다면, 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨겨라.
> -  경고를 숨기기로 한 근거를 주석으로 남겨라


[참조] (이펙티브 자바 3판)
