#### 멤버 클래스는 되도록 static으로 만들라

> 중첩클래스란?
> 다른 클래스 안에 정의된 클래스를 말한다.

중첩클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다.면 톱레벨 클래스로 만들어야 한다.

중첩클래스의 종류
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스

첫번재를 제외한 나머지는 내부 클래스(inner class)

정적 멤버 클래스
다른 클래스안에 선언되고 바깥 클래스의 private 멤버에도 접근할 수 있는 점만 제외하고 일반 클래스와 똑같다.
다른 정적 멤버와 똑같은 접근 규칙을 적용받는다.
private로 선언하면 바깥 클래스에서만 접근할 수 있다.
정적 멤버 클래스는 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다.
```java
public class Calculator {

    static enum  Operation{
        PLUS, MINUS
    }
}
public class Item24 {
    public static void main(String[] args) {
        System.out.println(Calculator.Operation.PLUS);
      }
}
```
결과
```java
PLUS
```

정적멤버 클래스와 비정적 멤버 클래스의 구문상의 차이는 static이 있고 없고 차이이지만, 의미상 차이는 꽤 크다.

비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
그래서 비정적 멤버 클래스의 인스턴스 메서드에서  정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.
```java
public class NotStaticClass {
    private int number = 1;

    public void test() {
      System.out.println("NotStaticClass : " + number);
    }

    //비정적 멤버 클래스
    public class MyClass {
      int myClassNumber = 2;

      public void myClassTopMemberTest() {
          myClassNumber = NotStaticClass.this.number;
          System.out.println("myClassNumber :" + myClassNumber);
          NotStaticClass.this.test();
      }
}


public class Item24 {
    public static void main(String[] args) {
      NotStaticClass notStaticClass = new NotStaticClass();
      notStaticClass.new MyClass().myClassTopMemberTest();
    }
}
```
결과

```java
myClassNumber :1
NotStaticClass : 1
```

정규화된 this란 클래스.this 형태로 바깥 클래스의 이름을 명시하는 용법을 말한다.
따라서 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤머 클래스로 만들어야 한다.
비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없기 때문이다.

비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화 될때 확립되며, 더 이상 변경할 수 없다.
이 관계는 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어지는게 보통이지만, 드물게 직접 바깥 인스턴스의 클래스.new MemberClass(args)를 호출해 수동으로 만들기도 한다.

```java
package effect.item24;

public class NotStaticClass {
    private int number = 1;

    public void test() {
        System.out.println("NotStaticClass : " + number);
    }

    public int getInnerClass() {
        return new MyClass().myClassNumber;
    }

    public class MyClass {
        int myClassNumber = 2;
        public void myClassTopMemberTest() {
            myClassNumber = NotStaticClass.this.number;
            System.out.println("myClassNumber :" + myClassNumber);
            NotStaticClass.this.test();
        }
        public void myClassTest() {
            NotStaticClass.this.number = myClassNumber;
            NotStaticClass.this.test();
        }
    }
}

public class Item24 {
    public static void main(String[] args) {
      NotStaticClass notStaticClass = new NotStaticClass();
      notStaticClass.test();  //바깥 인스턴스
      notStaticClass.new MyClass().myClassNumber = 4; //4로 접근불가능
      notStaticClass.new MyClass().myClassTest();
      System.out.println(notStaticClass.getInnerClass());
    }
}
```
결과
```java
NotStaticClass : 1
NotStaticClass : 2
2
```
이 관계의 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성 시간도 더 걸린다.
비정적 멤버 클래스는 어댑터를 정의할 때 자주 쓰인다.
즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것이다.
Map 인터페이스의 구현체들은 KeySet, entrySet, values 메서드가 반환하는 자신의 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용한다.
Set과 List같은 다른 컬렉션 인터페이스 구현들도 자신의 반복자를 구현할 때 비정적 멤버 클래스를 주로 사용한다.

```java
public class MySet<E> extends AbstractSet<E> {
    //...생략

    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }
    private class MyIterator implements Iterator<E> {
        //...생략
    }
}
```
ArrayList 예시
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
          // ...생략

    /**
     * Returns an iterator over the elements in this list in proper sequence.
     *
     * <p>The returned iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
     *
     * @return an iterator over the elements in this list in proper sequence
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * An optimized version of AbstractList.Itr
     */
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        // prevent creating a synthetic constructor
        Itr() {}

        // .. 생략
    }
```
**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.**
static를 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다.
이 참조를 저장하려면 시간과 공간이 소비된다. 더 심각학 문제는 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다는 점이다.


익명클래스
바깥 클래스의 멤버도 아니다. 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
코드의 어디서든 만들수 있다. 그리고 오직 비정적 문맥에서 사용될때만 바깥 클래스의 인스턴스를 참조할 수 있다.
정적 문맥에서라도 상수 변수 이외의 정적 멤버는 가질 수 없다.
즉, 상수 표현을 위해 초기화된 final 기본타입과 문자열 필드만 가질 수 있다.

익명 클래스는 제약이 많다. 선언한 지점에서만 인스턴스를 만들 수 있고, instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다. 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수 도 없다.
익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에 상속한 멤버 외에는 호출할 수 없다. 익명 클래스는 표현식 중간에 등장하므로 짧지만 가독성이 떨어진다.

자바 람다를 지원하기 전에는 즉석에서 작은 함수 객체나 처리 객체를 만드는데 익명 클래스를 주로 사용했다.
익명 클래스의 또 다른 주 쓰임은 정적 팩터리 메서드를 구현할 때다.
```java
package effect.item24;

public class HelloWorldAnonymousClasses {

    interface HelloWorld {
        void greet();
        void greetSomeone(String someone);
    }

    public void sayHello () {

        class EnglishGreeting implements HelloWorld {
            String name = "world";
            @Override
            public void greet() {
                greetSomeone("world");
            }

            @Override
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Hello, " + name);
            }
        }

        HelloWorld englishGreeting = new EnglishGreeting();

        //익명클래스
        HelloWorld frenchGreeting = new HelloWorld() {
            String name = "tout le monde";
            @Override
            public void greet() {
                greetSomeone("tout le monde");
            }

            @Override
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Hello, " + name);
            }
        };

        HelloWorld spanishGreeting = new HelloWorld() {
            String name = "mundo";
            public void greet() {
                greetSomeone("mundo");
            }
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Hola, " + name);
            }
        };

        englishGreeting.greet();
        frenchGreeting.greetSomeone("Fred");
        spanishGreeting.greet();
    }

    public static void main(String[] args) {
        HelloWorldAnonymousClasses hello = new HelloWorldAnonymousClasses();
        hello.sayHello();
    }
}
```
결과
```java
Hello, world
Hello, Fred
Hola, mundo
```
Comparator 예시
```java
List list = new ArrayList<Pair>();
list.add(Pair.of("B", 1));
list.add(Pair.of("A", 2));
list.add(Pair.of("C", 4));

Collections.sort(list);
System.out.println(list);

Collections.sort(list, new Comparator<Pair>() {
    @Override
    public int compare(Pair o1, Pair o2) {
        if ((Integer)o1.getRight() > (Integer)o2.getRight())
            return 1;
        else if ((Integer)o1.getRight() < (Integer)o2.getRight())
            return  -1;
        else
            return 0;
    }
});

System.out.println(list);
```
결과
```java
[(A,2), (B,1), (C,4)]
[(B,1), (A,2), (C,4)]
```
지역클래스
지역변수를 선언할 수 있는 곳이라면 실질적으로 어디서든 선언할 수 있고, 유효 범위도 지역변수와 같다.
다른 세 중첩 클래스와의 공통점도 하나씩 가지고 있다.
멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다.
익명 클래스처럼 비정적 문맥에서 사용될때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버를 가질 수 없으며, 가독성을 위해 짧게 작성해야 한다.

_핵점정리_
> - 중첩 클래스는 4가지가 있으며 각각 쓰임이 다르다.
> - 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다.
> - 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않다면 정적으로 만든다.
> - 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 시점에 단 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않으면 지역 클래스로 만든다.

[참조] (이펙티브 자바 3판)
