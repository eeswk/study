#### 태그 달린 클래스보다는 클래스 계층구조를 활용하라

> 두가지 이상의 의미를 표현할 수 있으며,
> 현재 표현하는 의미를 태그값으로 알려주는 클래스

태그 달린 클래스 - 하나의 클래스가 2가지 이상의 기능으로 동작할 때, 어떤 기능으로 동작할지를 필드값으로 구분하는 클래스

```java
package effect.item23;

public class Figure {
    enum Shape { RECTANGLE, CIRCLE};

    // 태그필드 - 현재 모양을 나태냄
    final Shape shape;
    // 사각형일때만 쓰인다.
    double length;
    double width;
    //원일때만 쓰인다.
    double radius;

    //원 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    //사각형 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```
태그 달린 클래스에는 단점이 많다.
1. 열거타입선언, 태그필드, switch문 등 쓸데 없는 코드가 많다.
2. 여러 구현이 한 클래스에 혼합되어 가독성이 나쁘다.
필드를 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야한다(쓰지 않는 필드를 초기화하는 불필요한 코드가 늘어난다.)

생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드를 초기화하는데 컴파일러가 도와줄 수 있는건 별로 없다. 엉뚱한 필드를 초기화해도 런타임에야 문제가 드러날뿐이다. 또 다른 의미를 추가하려면 코드를 수정해야 한다.

새로운 의미를 추가할때마다 모든 switch문을 찾아 처리하는 코드를 추가해야 하는데, 하나라도 빠뜨리면 역시 런타임에 문제가 발생한다.
인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.
**태그 달린 클래스는 장황하고 오류를 내기 쉽고, 비효율적이다.**
자바와 같은 객체지향 언어는 타입 하나로 다양한 의미의 객체를 표현하는 훤씬 나은 수단을 제공한다.
바로 클래스 계층 구조를 활용하는 서브타입핑(subtyping)이다.
태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류일뿐이다.

_태그달린 클래스를 클래스 계층구조로 바꾸는 방법_
1. 계층구조의 루트(root)가 될 추상 클래스를 정의한다.
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.(Figure클래스의 area()메서드)
3. 그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.(Figure 클래스에서는 태그값에 상관없는 메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드도 없다. 루트 클래스에는 추상 메서드 area 하나만 남게 된다.)
5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.(Figure를 확장한 원 Circle 클래스와 사각형 Rectangle 클래스를 만든다. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드들을 넣는다.)
6. 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다.

```java
public abstract class Figures {
    abstract double area();
}

public class Circle extends Figures {
    final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

public class Rectangle extends Figures {
    final double length;
    final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```
_장점_

- 클래스 계층 구조는 태그 달린 클래스의 단점을 모두 없앤다.
- 간결하고 명확하며, 쓸데 없는 코드는 사라졌다.
- 각 의미를 독립된 클래스에 담아 관련없는 데이터 필드를 모두 제거했다.
- 살아 남은 필드는 모두 final이다.
- 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다.
- 실수로 빼먹은 case문 때문에 런타임 오류가 발생 할일도 없다.
- 루트 클래스의 코드를 건드리지 않고도 다른 프로그래머들이 독립적인 계층 구조를 확장하고 함께 사용할 수 있다.
- 타입의 의미별로 따로 존재하니 변수의 의미를 명시하거나 제한할 수 있다.
- 특정의미만 매개변수로 받을 수 있다.
- 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일 타임 타입 검사 능력을 높여준다.

정사각형을 추가할경우 간단하게 반영할 수 있다.
```java
public class Square extends Rectangle {
    public Square(double side) {
        super(side, side);
    }
}
```
단, 접근자 메서드 없이 필드를 직접 노출했다. 이는 단지 코드를 단순하게 처리용으로 사용했다. 공개할 클래스라면 이렇게 설계하는것은 좋지 않다.

_핵심정리_

>  - 태그달린 클래스를 써야 하는 상황은 거의 없다.
> - 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층 구조로 대체하는 방법을 생각해보자.
> - 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩토링하는 걸 고민해보자.

[참조] (이펙티브 자바 3판)
