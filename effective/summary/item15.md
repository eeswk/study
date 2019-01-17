#### 클래스와 멤버의 접근 권한을 최소화

_잘 설계된 컴포넌트의 특징_

* 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨김.
*  즉 오직 API를 통해서만 다른 컴포넌트와 소통.

_정보은닉 장점_

* 시스템 개발 속도 높임. (여러 컴포넌트를 병렬로 개발 가능)
* 시스템 관리비용 낮춤. (각 컴포넌트를 더 빨리 파악, 디버깅, 교체)
* 소프트웨어 재사용성 높임. (외부에 의존하지 않고, 독자적으로 쓰일 가능성 높임 )
* 큰 시스템을 제작하는 난이도를 낮춤. (시스테 전체가 완성되지 않는 상태에서도 개별 컴포넌트의 동작을 검증 가능)

_접근제한자를 활용하는 것이 정보 은닉의 핵심_

기본원칙 - 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.

- private -멤버를 선언한 톱레벨 클래스에서만 접근 가능
- package-private - 멤버가 소속된 패키지 안의 모든 클래스에서 접근 가능
- protected - 선언한 크래스의 하위 클래스에서도 접근 가능
- public - 모든 곳에서 접근 가능

How?
1. 클래스의 공개 API를 세심히 설계한 후, 그 외의 모든 멤버는 private으로 만든다.
2. 오직 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 대하여 package-private으로 풀어준다.
3. 권한을 풀어 주는일을 자주 하면 시스템에서 컴포넌트를 더 분해해야 하는지 다시 고민한다.

tip

클래스에서 public static final 배열필드 또는 필드를 반환하는 접근자 메서드를 제공해서는 안된다.

```java
public static final Thing[] VALUES = {...};
```

1. public 배열을 private로 만들고 public 불변 리스트를 추가
```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> VALUES = Collections.unmodifiableList<Arrays.asList(PRIVATE_VALUES));
```

2. private으로 만들고 그 복사본을 반환하는 public 메서드를 추가
```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final Thing[] values() {
  return PRIVATE_VALUES.clone();
}
```

예제
```java
public class Item15 {
    public static final String[] VALUES = {"1", "2", "3", "4"};

    private static final String[] PRIVATE_VALUES = {"1", "2", "4", "3"};

    //읽기전용
    public static final List<String> LIST_VALUES =
            Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

    //정렬가능
    public static final List<String> LIST_VALUES_SORT =
            Arrays.asList(PRIVATE_VALUES);

    //불변
    //연결이 끊어지고 요소 만 공유합니다.
    public static final List<String> LIST_VALUES_NOT_MODIFY =
            Collections.unmodifiableList(new ArrayList<>(LIST_VALUES));

}
```
```java
public class Item15TestDriven {

    public static void main(String[] args) {
        Item15 item15 = new Item15();

        String[] list1 = item15.VALUES;
        //Error:(14, 32) java: PRIVATE_VALUES has private access in effect.Item15
        //접근안됨
        //String[] list2 = item15.PRIVATE_VALUES;
        List<String> list3 = item15.LIST_VALUES;
        List<String> list4 = item15.LIST_VALUES_SORT;
        List<String> list5 = item15.LIST_VALUES_NOT_MODIFY;

        //final 선언인데 수정 가능
        item15.VALUES[0] = "5";

        for (String s: list1) {
            System.out.println(s);
        }

//        for (String s: list2) {
//            System.out.println(s);
//        }
        System.out.println("===");
        for (String s: list3) {
            System.out.println(s);
        }
        //정렬
        Collections.sort(list4, Comparator.comparing(Object::toString).reversed());
        System.out.println("===");
        for (String s: list4) {
            System.out.println(s);
        }
        System.out.println("===");
        for (String s: list3) {
            System.out.println(s);
        }
        System.out.println("===");
        for (String s: list5) {
            System.out.println(s);
        }
    }
}
```
예제결과
~~~java
5
2
3
4
===
1
2
4
3
===
4
3
2
1
===
4
3
2
1
===
1
2
4
3
~~~

[참조] (이펙티브 자바 3판)

[참조] (http://www.javacreed.com/modifying-an-unmodifiable-list/)
