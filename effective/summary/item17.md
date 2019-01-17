#### 변경 가능성을 최소화하라

_불변클래스_

인스턴스의 내부 값을 수정할 수 없는 클래스

_특징_

* 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.

* 스레드에 안전하여 따로 동기화할 필요가 없다.

_예_

String, 기본타입의 박싱된 클래스들, BigInteger, BigDecimal

_불변 클래스 규칙_
* 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
* 클래스를 확장할 수 없도록 한다. (클래스를 final로 선언)
* 모든 필드를 final로 선언
* 모든 필드를 private로 선언
* 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

~~~java
//불변 클래스
package effect;

public final class Item17 {
    private final double re;
    private final double im;

    public Item17(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Item17 plus(Item17 c) {
        return new Item17(re + c.re, im +c.im);
    }
    public Item17 minus(Item17 c) {
        return new Item17(re - c.re, im - c.im);
    }

    public Item17 times(Item17 c) {
        return new Item17(re * c.re - im * c.im,
                re * c.im + im*c.re);
    }

    public Item17 dividedBy(Item17 c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Item17((re * c.re + im * c.im)/ tmp,
                (im * c.re - re *c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;

        if (!(o instanceof  Item17))
            return false;
        Item17 c = (Item17) o;

        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" +
                "re=" + re +
                ", im=" + im +
                ")";
    }
}

~~~

_설명_
1. plus, minus, times, dividedBy 사칙연산 메서들이 인스턴스 자신은 수정하지 않고 새로운 인스턴스를 만들어 반환한다.
(이 방식으로 프로그래밍 하면 코드에서 불변이 되는 영역의 비율이 높아지는 장점을 누릴 수 있다.)

2. 불변 객체는 단순한다.
(불변객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
모든 생성자가 불변식(class invariant)을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다.)

3. 스레드 안전하다.
(여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.)

4. 안심하고 공유할 수 있다.
(불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기 권한다.)
~~~java
public static final Item17 ZERO = new Item17(0, 0);
public static final Item17 ONE = new Item17(1, 0);
public static final Item17 I = new Item17(0, 1);
~~~

_더 깊게_

불변클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 적정 팩토리를 제공할 수 있다.
박싱된 기본 타입 클래스 전부와 BigInteger
정적 팩토리를 사용하면 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.
새로운 클래스를 설계할때 public 생성자 대신 정적 팩토리를 만들어두면, 클라이언트를 수정하지 않고도 필요에 따라 캐시 기능을 나중에 덧붙일 수 있다.
방어적 복사도 필요없고 따라서 clone 메서드나 복사 생성자를 제공하지 않는게 좋다.
불변객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
객체를 만들때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
불변객체는 그 자체로 실패 원자성을 제공한다.


~~~java
//생성자 대신 정적 팩터리르 사용한 불변 클래스

public class Item17 {
  private final double re;
  private final double im;

  private Item17(double re, double im) {
    this.re = re;
    this.im = im;
  }

  public static Item17 valueOf(double re, double im) {
    return new Item17(re, im);
  }
}
~~~

_핵심정리_

게터가 있다고 해서 무조건 세터를 만들지 말자.
클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
객체가 가질 수 있는 상태의 수를 줄이면 그 객체를 예측하기 쉬워지고 오류가 생길 가능성이 줄어든다.
꼭 변경해야 할 필드를 뺀 나머지 모두를 final로 선언하자.
다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.
생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.
확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해서는 안된다.
객체를 재활용할 목적으로 상태를 다시 초기화하는 메서드도 안된다.
복잡성만 커지고 성능 이점은 거의 없다.
