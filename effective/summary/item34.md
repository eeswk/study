#### int 상수 대신 열거타입을 사용하라
열거타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.\
예) 봄, 여름, 가을, 겨울

열거 타입 지원하기 전
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;
```

정수 열거 패턴 기법의 단점
- 타입 안정을 보장할 방법이 없고, 표현력도 좋지 않다.
- 별도 이름 공간을 지원하지 않기 때문에 접두어를 써서 이름 충돌을 방지하는 것이다.
- 깨지기 쉽다. (평범한 상수를 나열한것 뿐. 컴파일하면 그값이 클라이언트 파일에 그대로 새겨진다. 따라서 다시 컴파일)
- 정수 상수는 문자열로 출력하기 까다롭다.(단지 숫자로 보임)
- 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법도 마땅치 않다.
- 그안에 상수가 몇개 있는지 알수 없다.

그래서 정수 대신 문자열 상수를 변경하는 패턴도 있다.\
문자열 열거 패턴이라는 하는 이 변경은 더 나쁘다.

대안은 열거 타입(Enum type)이다.

 ```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
 ```

자바의 열거 타입은 완전한 형태의 클래스라서 (단순한 정수 값일뿐) 다른 언어의 열거 타입보다 훨씬 강력하다.\
열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.\
열거타입은 밖에서 접근할수 있는 생성자를 제공하지 않으므로 사실상 final이다.\
따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할수 없으니 열거 타입 선언ㅇ로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.

- 인스턴스 통제
- 싱글턴은 원소가 하나뿐인 열거타입이라고 할수 있다.
- 거꾸로 열거 타입은 싱글턴을 일반화한 형태

컴파일타임 타입 안정성을 제공.\
임의의 메서드나 필드를 추가할수 있고, 임의의 인터페이스를 구현하게 할수도 있다.

데이터와 메서드를 갖는 열거 타입
```java
package effetive.item34;

public enum Planet {
    MERCYRYT(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6);

    private final double mass;              //질량(단위: 킬록램)
    private final double radius;            //반지름(단위: 미터)
    private final double surfaceGravity;    //표면중력(단위: m /s^2)

    //중력상수(단위:m^3 / kg S^2)
    private static final double G = 6.67300E-11;

    //생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {return mass;}
    public double radius() {return radius;}
    public double surfaceGravity () {return  surfaceGravity;}

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```
열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면된다.\
열거 타입은 근본적으로 불변이라 모든 필드는 final이어야 한다.\
필드를 public으로 선언해도 되지만 private로 두고 별도의 public 접근자 메서드를 두는게 낫다.

Planet의 생성자에서 표면중력을 계산해 저장한 이유는 단순히 최적화를 위해서다.
 ```java
 package effetive.item34;

public class WeightTable {
    public static void main(String[] args) {
        double earthWeight = Double.parseDouble("185");
        double mess = earthWeight / Planet.EARTH.surfaceGravity();
        for(Planet p : Planet.values()) {
            System.out.printf("%s에서의 무게는 %f이다.\n",  p, p.surfaceWeight(mess));
        }
    }
}
 ```
 결과
 ```
MERCYRY에서의 무게는 19.785863이다.
VENUS에서의 무게는 47.385282이다.
EARTH에서의 무게는 185.000000이다.
 ```
열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적메서드인 values를 제공한다. 값들은 선언된 순서로 저장된다.\
각 열거 타입 값의 toString 메서드는 상수 이름을 문자열로 반환하므로 println과 printf로 출력하기 좋다.\
기본 toString이 제공하는 이름을 재정의할수 있다.

열거타입에서 상수를 제거한다면? 아무 영향이 없다.\
열거타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private나 package-private 메서드로 구현한다.\
이렇게 구현된 열거 타입 상수는 자신을 선언한 클래스 혹은 패키지에서만 사용할수 있는 기능을 담게된다.

열거타입은 상수별로 다르게 동작하는 코드를 구현
```java
package effetive.item34;

public enum Operation {
    PLUS { public  double apply(double x, double y) { return  x + y; }},
    MINUS { public  double apply(double x, double y) { return  x - y; }},
    TIMES { public  double apply(double x, double y) { return  x * y; }},
    DIVIDE { public  double apply(double x, double y) { return  x / y; }};

    public abstract double apply(double x, double y);
}

```

상수별 클래스몸체(class body)와 데이터를 사용한 열거타입
```java
package effetive.item34;

public enum OperationBody {
    PLUS("+") {
        public  double apply(double x, double y) { return  x + y; }
    },
    MINUS("-") {
        public  double apply(double x, double y) { return  x - y; }
    },
    TIMES("*") {
        public  double apply(double x, double y) { return  x * y; }
    },
    DIVIDE("/") {
        public  double apply(double x, double y) { return  x / y; }
    };

    private final String symbol;

    OperationBody(String symbol) { this.symbol = symbol; }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);

    public static void main(String[] args) {
        double x = Double.parseDouble("2.000000");
        double y = Double.parseDouble("4.000000");
        for (OperationBody o : OperationBody.values())
            System.out.printf("%f %s %f = %f\n" , x, o, y, o.apply(x, y));
    }
}

```
결과
```
2.000000 + 4.000000 = 6.000000
2.000000 - 4.000000 = -2.000000
2.000000 * 4.000000 = 8.000000
2.000000 / 4.000000 = 0.500000
```
열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동으로 생성된다.\
열거 타입의 toString 메서드를 재정의하려든, toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 걸 고려해보자.

열거타입용 fromString 메서드
```java
private static final Map<String, OperationBody> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e -> e));

//지정한 문자열에 해당하는 OperationBody가 존재하면 반환한다.
public static Optional<OperationBody> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```
OperationBody 상수가 stringToEnum 맵에 추가되는 시점은?\
열거타입 상수 생성후 정적 필드가 초기화 될 때이다.\
values 메서드가 반환하는 배열 대신 스트림을 사용했다.\
자바 8 이전에는 빈 해시맵을 만든 다음 values가 반한환 배열을 순회하며{문자열, 열거타입 상수} 쌍을 맵에 추가했을 것이다.\
하지만 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다.\
이렇게 하려면 컴파일 오류가 난다. 이 방식을 허용했다면 런타임에 NullPointerException이 발생할 것이다.\
열거 타입의 정적 필드 중 열거 타입의 생성자에게 접근할수 있는 것은 상수 변수 뿐이다.\
열거 타입 생성자가 실행되는 시점에는 정적 필드들이 아직 초기화되기 전이라, 자기 자신을 추가하지 못하게 하는 제약이 꼭 필요하다.\
이 제약의 특수한 예로, 열거 타입 생성자에게 같은 열거 타입의 다른 상수에도 접근할수 없다.\

fromString이 Optional<OperationBody>을 반환 부분\
문자열이 가리키는 연산이 존재하지 않을 수 있음을 클라이언트에게 알리고, 그 상황을 클라이언트에서 대처하도록 한것이다.\

값에 따라 분기하는 코드를 공유하는 열거타입- 좋은 방법?
```java
package effetive.item34;

public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

     int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch (this) {
            case SATURDAY: case SUNDAY: //주말
                overtimePay = basePay / 2;
                break;
            default: //주중
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                  0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2 ;
        }
        return basePay + overtimePay;
    }

    public static void main(String[] args) {
         int minutesWorked = 480, payRate = 2;
        System.out.println(PayrollDay.MONDAY.pay(minutesWorked, payRate));
        System.out.println(PayrollDay.SATURDAY.pay(minutesWorked, payRate));
        System.out.println(PayrollDay.SUNDAY.pay(minutesWorked, payRate));
    }
}
```
관리문제- 휴가와 같은 새로운 열거 타입이 추가될경우
1. 잔업수당을 계산하는 코드를 모두 상수에 중복해서 넣는다.
2. 계산 코드를 평일용과 주말용으로 나눠 각각 도우미 메서드를 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출하면된다.
3. 가장 깔끔한 방법\
새로운 상수를 추가할때 잔업수당'전략'을 선택하도록 하는것이다.\
잔업수당 계산을 private 중첩 열거 타입으로 옮기고 PayrollDay 열거 타입의 생성자에서 이중 적당한 것을 선택한다.\
그러면 PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, switch문이나 상수별 메서드 구현이 필요없게 된다.\
이 패턴은 switch문보다 복잡하지만 더 안전하고 유연하다.

전략 열거 타입 패턴
```java
package effetive.item34;

public enum PayrollDay2 {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay2(PayType payType) { this.payType = payType; }
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <=  MINS_PER_SHIFT ?
                0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2 ;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate;
            }
        };
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate ) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked, payRate);
        }
    }

    public static void main(String[] args) {
        int minutesWorked = 480, payRate = 2;
        System.out.println(PayrollDay2.MONDAY.pay(minutesWorked, payRate));
        System.out.println(PayrollDay2.SATURDAY.pay(minutesWorked, payRate));
    }
}
```
열거 타입의 성능은 정수 상수와 별반 다르지 않다.\
열거 타입을 메모리에 올리는 공간과 초기화하는 시간이 들지만, 체감된 정도는 아니다.

열거 타입을 과연 언제 쓰나?\
필요한 원소를 컴파일타임에 다 알수 있는 상수 집합이라면 항상 열거 타입을 사용하자.\
열거타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.\
나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다.

_핵심정리_
> - 열거 타입은 정수 상수보다 뛰어나다.
> - 읽기 쉽고 안전하고 강력하다.
> - 열거 타입이 명시적 생성자나 메서드 없이 쓰지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작할때는 필요하다.
> - 하나의 메서드가 상수별로 다르게 동작해야 할때도 있다. switch문 대신 상수별 메서드 구현을 사용하자.
> - 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.


[참조] (이펙티브 자바 3판)
