#### public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

_객체지향 프로그래머_
* 필드를 모두 private로 public 접근자(getter)를 사용
~~~java
class Point {
  public double x;
  public double y;
}
~~~
_이유?_
* 캡슐화의 이점을 제공하지 못함
* API를 수정하지 않고는 내부 표현을 바꿀수 없음
* 불변식을 보장할 수 없음
* 외부에서 필드에 접근할때 부수 작업을 수행할 수도 없음

~~~java
class Point {
  private double x;
  private double y;

  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }
  public void setX(double x) { this.x = x }
  public void setY(double y) { this.y = y }
}
~~~

_이점?_

패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 바꿀수 있는 유연성을 얻을 수 있다.

_핵심정리_

> - public 클래스는 절대 가변 필드를 직접 노출해서는 안된다.
> - 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
> - 하지만 package-private 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.

[참조] (이펙티브 자바 3판)
