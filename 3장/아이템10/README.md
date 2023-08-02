# 아이템 10. equals는 일반 규약을 지켜 재정의하라

두 객체가 동일한지 판단할 때 `equals()` 메서드를 재정의 해 사용하곤 한다. 하지만 equals 메서드는 재정의 할 때 고려해야할 것들이 많아서, 잘못하면 디버깅이 어려운 오류가 터질 수 있다. 그렇기 때문에 equals 메서드를 재정의하지 않고 오직 자기 자신과 같도록 놔두는 게 좋을 수도 있다.

그렇다면 equals 메서드는 어느 경우에 재정의 하는게 좋고, 또 어느 경우에 재정의 하지 않는게 좋을까?

### 재정의 하지 않는 경우

* 각 인스턴스가 본질적으로 고유한 경우
  
  값을 가지고 있는 객체가 아닌, 동작만을 위한 객체를 표현하는 클래스인 경우가 여기 속한다. <br>
  예) Thread

* 같은 값의 인스턴스가 둘 이상 만들어지지 않는 것이 보장될 경우

  값을 가지고 있어도, 인스턴스가 하나밖에 만들어지지 않는 클래스들이다.<br>
  예) Enum

* 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없는 경우

  클래스 설계시 논리적 동치성을 검사하기 원하지 않거나 필요 없다고 판단될 경우이다. <br>
  예) java.util.regex.Pattern 

* 상위 클래스에서 재정의한 equals가 하위 클래스에서 그대로 사용될 수 있는 경우

  예) Set, List, Map

* 클래스가 private이거나 package-private이라 equals 메서드를 호출할 일이 없는 경우

추가적으로 equals 메서드를 완전히 사용하지 못하도록 막고 싶다면, 아래의 코드처럼 예외를 발생시키게 할 수도 있다.

```JAVA
public class NoEquals {
    ...
    @Override
    public boolean equals(Object o) {
      throw new AssertionError(); // 호출 금지!
    }
}
```

### 재정의 하는 경우

* 객체의 '식별성'이 아닌 '논리적 동치성'으로 판단해야하는 경우

  객체의 값을 비교해야 하는 클래스가 여기 속한다.<br>
  ex) Integer, String 등, 값을 가지는 클래스

## equals 메서드 재정의 규약

다음은 equals 메서드를 재정의할 때 이상 현상을 방지하기 위해 Object 명세에 적혀있는 규약들이다.

### 반사성

> null이 아닌 모든 참조값 x에 대해, x.equals(x) 는 true다.

단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻이다.

### 대칭성

> null이 아닌 모든 참조값 x, y에 대해, x.equals(y)가 true면, y.equals(x)도 true다.

대칭성은 두 객체가 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다.

다음은 대칭성을 위배한 코드의 예이다.

```JAVA
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override 
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
                    
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}
```

위 코드는 언뜻 보면 올바른 코드 같지만, 아래의 경우에 대칭성이 위반되게 된다.

```JAVA
public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Hello");
        String s = "hello";
        System.out.println(cis.equals(s)); // true
        System.out.println(s.equals(cis)); // false
}
```

String 에서는 CaseInsensitiveString 클래스를 모르기 때문에 이런 상황이 발생하게 된다.

이 문제를 해결하기 위해서는 상위 클래스와의 연동을 포기해야 한다.

```JAVA
@Override 
public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

### 추이성

> null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true이다.

추이성은 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다.

해당 속성은 **상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하는 상황에서 어기기 쉽다**.

다음과 같은 2차원 표면의 좌표를 표현하는 클래스가 있다고 하자.

``` JAVA
public class Point {  // 부모 클래스
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}
```

이 클래스를 확장해 색상을 더한다고 해보자.

```JAVA
public class ColorPoint extends Point {  // 자식 클래스
	private final Color color;
    
    public ColorPoint(int x, int y, Color color){
    	super(x,y);
        this.color = color;
    }
}
```

이 상황에서 equals 메서드는 어떻게 구현해야 할까?

1. 좌표와 색상 모두를 비교하게 구현하기

    ```JAVA
    public class ColorPoint extends Point {
        ...
      
        @Override 
        public boolean equals(Object o) {
            if (!(o instanceof ColorPoint))
                return false;
            return super.equals(o) && ((ColorPoint) o).color == color;
        }

        //대칭성 위배
        public static void main(String[] args) {
            Point p = new Point(1, 2);
            ColorPoint cp = new ColorPoint(1, 2, Color.RED);
            System.out.println(p.equals(cp)); //true
            System.out.println(cp.equals(p)); //false
        }
    }
    ```

    Point 클래스에선 좌표만 비교해 true가 나오고, ColorPoint 클래스에선 색상이 달라 false 가 나오므로, 대칭성이 위배된다.

2. ColorPoint 에서 Point와 비교할 때 색상을 무시하도록 구현하기

    ```JAVA
    public class ColorPoint extends Point {
        ...
      
        @Override 
        public boolean equals(Object o) {
            if (!(o instanceof Point))
            return false;

            //o가 일반 Point면 색상을 무시하고 비교한다.
            if (!(o instanceof ColorPoint))
              return o.equals(this);

            // o가 ColorPoint면 색상까지 비교한다.
            return super.equals(o) && ((ColorPoint) o).color == color;
        }

        //추이성 위배
        public static void main(String[] args) {
            ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
            Point p2 = new Point(1, 2);
            ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
            System.out.println(p1.equals(p2)); //true
            System.out.println(p2.equals(p3)); //true
            System.out.println(p1.equals(p3)); //false
        }
    }
    ```

    p1과 p2, p2와 p3는 색상을 무시하고 좌표만 비교하므로 true 가 나오지만, p1과 p3은 다른 색을 가지고 있어 false 가 나오므로, 추이성이 위배된다.

3. Point와 ColorPoint의 비교가 무조건 false가 나오도록, 클래스를 비교하도록 구현하기

    ```JAVA
    public class Point {
        ...
      
        @Override 
        public boolean equals(Object o) {
            if (o == null || o.getClass() != getClass())
              return false;
            Point p = (Point) o;
            return p.x == x && p.y == y;
        }
    }
    ```

    이렇게 되면 대칭성, 추이성 모두 지켜지게 되지만, 이번에는 다른 곳에서 문제가 발생한다. 

    ColorPoint 가 Point를 상속받은 이상, SOLID 원칙이 지켜져야 하는데, 이 중 LSP를 위배하게 되는 것이다. 
    
    LSP, 즉 리스코프 치환 원칙은 어떤 클래스의 모든 메서드는 그 하위의 클래스에서도 똑같이 잘 작동해야 한다는 원칙이다. 이에 따르면 Point의 하위 클래스인 ColorPoint는 정의상 여전히 Point 이므로, 어디서든 Point로써 활용될 수 있어야 한다.

    하지만 우리는 방금 위의 equals 메서드로 'Point와 ColorPoint는 서로 다른 것' 이라고 선언했으므로, ColorPoint는 Point로써 사용될 수 없게 되어버렸다. 이 오류가 잘 드러나는 부분은 컬렉션의 `.contains()` 메서드이다.

    ```JAVA
        public static void main(String[] args) {
            Set<Point> points = Set.of(
                new Point(1, 2)
            );

            ColorPoint p1 = new ColorPoint(1, 2, Color.RED);

            points.contains(p1); // false
        }
    ```

    contains 메서드는 내부적으로 equals 를 이용해 포함 여부를 반환한다. ColorPoint가 Point로도 사용될 수 있다면, Point 클래스 기준으로 같은 값을 가진 ColorPoint라면 컬렉션 포함 여부가 참이 나와야 하지만, 그렇지 못하고 있다. 즉, LSP가 위배되도록 구현된 equals 메서드 때문에 Point로 사용될 수 없게 되었다.

4. 상속을 포기하고 컴포지션을 사용해 구현하기

    결론부터 말하자면, 이 방법이 가장 규약을 잘 지킨 방법이다.

    우리의 목적은 '특정 클래스를 확장해 새로운 값을 추가하면서도, equals 규약을 만족시키는 방법' 을 찾는 것이었다. 하지만 이를 '객체 지향적 추상화'의 이점을 가진 채로 구현하는 방법은 존재하지 않는다.

    만약 추상화를 포기하기로 한다면, 컴포지션을 사용해 다음과 같이 구현할 수 있다.

    ```JAVA
    public class ColorPoint {
        private final Point point;  // 컴포지션
        private final Color color;

        public ColorPoint(int x, int y, Color color) {
            point = new Point(x, y);
            this.color = Objects.requireNonNull(color);
        }

        public Point asPoint() {  // 뷰 반환 메서드
            return point;
        }

        @Override public boolean equals(Object o) {
            if (!(o instanceof ColorPoint))
                return false;
            ColorPoint cp = (ColorPoint) o;
            return cp.point.equals(point) && cp.color.equals(color);
        }

        @Override public int hashCode() {
            return 31 * point.hashCode() + color.hashCode();
        }
    }
    
    ```

> **📌 참고**<br>
> `java.sql.Timestamp` 는 `java.util.Date`를 확장한 후 nanoseconds 필드를 추가해 구현되어 있다.<br>
> 이 탓에 Timestamp 클래스의 equals 메서드는 대칭성을 위반하게 되었고, 상위 클래스인 Date와 혼용하면 이상 현상이 일어날 수 있어, 이는 API 설명에도 적혀있다. <br>
> 따라서 이 두 클래스를 혼용하지 않도록 하자.

### 일관성
- 일관성이란 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻이다.
- 가변 객체는 시점에 따라 같을 수도, 다를 수도 있지만 불변 객체는 한번 다르면 끝까지 달라야 한다.
  
#### 클래스가 불변이든 가변이든 `equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.`
- 즉 equals에는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

    > 📌 **만약 신뢰할 수 없는 외부 자원인 URL이 equals 판단에 포함된다면?**
    >
    > - java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다.
    > - 그런데 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하는데 그 결과가 항상 같다고 할 수 없다.
    >   - 왜냐하면 유동 IP가 될 수도 있기 때문이다.

### 모든 객체가 null과 같지 않도록 equals를 재정의하라.
- 이는 equals의 판단에 사용되는 모든 필드나 객체가 null이 아닌 경우에 동일하다고 판단해야 한다는 것이다.
- 간단하게 말하면 equals로 비교할 때 `o != null` 임을 체크해줘야 한다는 것이다.
- 또 equals를 판단할 때 클래스의 타입이 같은지도 체크해야 한다.
 
    ```java
    @Override
    public boolean equals(Object o) {
        // null을 명시적으로 체크한다.
        // 그러나 instancOf를 사용하면
        // 굳이 null 체크를 할 필요가 없다.
        if(o == null) {
            return false;
        }
    }

    @Override
    public boolean equals(Object o) {
        // instanceOf로 타입 검사와 동시에
        // null인 경우 false를 반환한다.
        if(!(o instanceOf MyType)) {
            return false;
        }
    }
    ```
<br>

## equals 메서드 구현 방법을 단계별로 정리해보기
### 1. `== 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.`
- 이는 단순한 성능 최적화용으로 비교 작업이 복잡한 상황일 때 값어치를 한다. (early return pattern)

    ```java
    @Override
    public boolean equals(Object o) {
        if (this == o)
            // 여기서 자기 자신의 참조이면 equal하다고 판단한다!
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        MyClass that = (MyClass)o;
        return Objects.equals(name, that.name) && Objects.equals(address, that.address);
    }
    ```

    [early return에 관한 고찰](https://thearchivelog.dev/article/are-early-returns-any-good/)

<br>

### 2. `instancof 연산자로 입력이 올바른 타입인지 확인한다.`
  - 이 때의 올바른 타입은 equals가 정의된 클래스인 것이 보통이지만, 가끔은 그 클래스가 구현한 특정 인터페이스가 될 수도 있다. 어떤 인터페이스는 자신을 구현한 서로 다른 클래스끼리도 비교할 수 있도록 equals 규약을 `수정`하기도 한다. 예를들어 Set, List, Map, Map.Entry 등이 있다.

    ```java
    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        // if (o == null || getClass() != o.getClass())
        if (!(this instanceof MyClass)) // 이 때 instancof로 타입체크를 한다!
            return false;
        MyClass that = (MyClass)o;
        return Objects.equals(name, that.name) && Objects.equals(address, that.address);
    }

    ```
    
    ```java
    // IDE에서 자동 생성해주는 방식
    if (o == null || getClass() != o.getClass()) {
        return false;
    }

    // 다형성인 경우에도 올바른 타입인지 확인하는 방식
    if (o instanceOf Parent) {
        return false;
    }

    ```
    > 📌 **getClass()와 instanceOf의 개인적 의견**
    > 
    > - if(o == null || getClass() != o.getClass())를 쓰면 정확히 그 타입만 허용한다는 것을 나타낸다.
    >   - 그러나 명시적으로 null 체크를 해줘야 한다.
    > - if(o instanceOf Parent)를 쓰면 다형성인 경우에도 허용한다는 것을 나타낸다.
    >   - null 처리가 묵시적으로 되지만, getClass()를 사용하는 것보다 더 넓은 범위를 허용한다.
    > - 그러니 상황에 맞게 고쳐서 쓰자!

<br>

### 3. `입력을 올바른 타입으로 변환한다.`
  - 앞선 2번에서 올바른 타입인지 체크했으므로 이 단계는 100% 성공한다.
  
    ```java
    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
         if (o == null || getClass() != o.getClass())
            return false;
        MyClass that = (MyClass)o; // 이 때 올바른 타입으로 변환한다!
        return Objects.equals(name, that.name) && Objects.equals(address, that.address);
    }

    ```

<br>

### 4. `핵심 필드들이 모두 일치하는지 하나씩 검사한다.`
  - 모든 필드가 일치하면 true를, 아니면 false를 반환한다.

  - 2단계에서 인터페이스를 사용했다면 필드 값을 가져올 때도 인터페이스의 메소드를 사용해야 한다. (구현체가 가진 고유의 메소드를 사용하지 말자!)
  - 타입별 비교하는 방법
    - float와 double 같은 실수형을 제외한 기본 타입은 `== 으로 비교하자.`
  
    - 참조타입 필드는 각각의 `equals로 비교하자.`
    - float는 Float.compare(float, float)로 비교하자.
    - double은 Double.compare(double, double)로 비교하자.
      - 실수인 경우 정적 메소드로 비교하는 이유는 Float.NaN, -0.0f 같은 특수한 부동 소수값을 다뤄야 하기 떄문이다.
      - 그리고 Float.equals나 Double.equals은 오토박싱을 수반할 수 있으므로 성능 상 좋지 않다.
    - 배열은 앞선 지침대로 한다.
      - 만약 배열의 모든 원소가 핵심필드라면 Arrays.equals()도 가능하다.

<br>

### 5. 전형적인 equals 메서드의 예시

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(short areaCode, short prefix, short lineNum) {
        ...
    }

    // 전형적인 equals 메서드 예시
    @Override
    public boolean equals(Object o) {
        // 1. 자기 자신을 참조한 것인지 확인한다.
        if(o == this){
            return true;
        }

        // 2. 올바른 타입인지 확인한다.
        if(!(o instanceof PhoneNumber)){
            return false;
        }

        // 3. 올바른 타입으로 형변환을 한다.
        PhoneNumber pn = (PhoneNumber)o;

        // 4. 각 핵심 필드를 equals로 판단한다.
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}
```
<br>

### 6. 정리
- 꼭 필요한 경우가 아니면 equals를 재정의하지 말자!