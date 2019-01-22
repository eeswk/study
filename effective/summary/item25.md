#### 톱레벨 클래스는 한 파일에 하나만 담으라

> 소스 파일 하나에 톱레벨 클래스가 여러개 선언 시
> 심각함 위험을 감수 해야하는 행위다.

한 클래스에 여러가지로 정의할 수 있으며, 그 중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일 한것에 따라 달라지기 때문이다.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```
```java
Utensil.java

class Utensil {
    static final String NAME = "pan";
}
class Dessert {
    static final String NAME = "cake";
}
```
결과
```java
pancake
```
우연히 똑같은 두 클래스를 담은 Dessert.java라는 파일을 생성
```java
Dessert.java

class Utensil {
    static final String NAME = "pot";
}
class Dessert {
    static final String NAME = "pie";
}
```
컴파일시
```java
javac Main.java Dessert.java
```

컴파일 오류가 나고 Utensil과 Dessert 클래스 중복 정의했다고 알려준다.

컴파일러는 가장 먼저 Main.java를 컴파일하고 그 안에서 Dessert 참조보다 먼저 나오는 Utensil 참조를 만나면 Utensil.java 파일을 살펴 Utensil과 Dessert를 모두 찾아낼 것이다.
그런 다음 컴파일러가 두번째 명령줄 인수로 넘어온 Dessert.java를 처리하려 할때 같은 클래스가 정의가 이미 있음을 알게 된다.

javac Main.java나 javac Main.java Utensil.java 명령으로 컴파일 하면 Dessert.java 파일을 작성하기 전처럼 pancake를 출력한다.
그러나 javac Dessert.java Main.java 명령으로 컴파일하면 potpie를 출력한다.
이처럼 컴파일러 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시 잡아야 할 문제이다.

해결책

톱레벨 클래스들(Utensil과 Dessert)을 서로 다른 소스 파일로 분리하면 된다.
굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하는 방법을 고민해볼 수 있다.
다른 클래스에 딸린 부차적인 클래스라면 정적 멤버 클래스로 만드는 쪽이 일반적으로 더 나을 것이다.
읽기 좋고, private으로 선언하면 접근범위도 최소로 관리할 수 있기 때문이다.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```


_핵심정리_
> - 소스 파일 하나에 반드시 톱레벨 클래스(톱레벨 인터페이스)를 하나만 담자.


[참조] (이펙티브 자바 3판)
