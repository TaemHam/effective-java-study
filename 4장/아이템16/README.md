# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 접근자 메서드란?
- 접근자 메서드란 클래스가 가지고 있는 필드의 값을 수정하거나, 값을 가져올 때 사용하는 메서드를 의미한다.

- 접근자 메서드는 개발자가 원하는 대로 만들어도 되지만 자바에서는 setter, getter라는 관례적인 메서드를 사용하고 있다.

## 왜 public 클래스에서는 접근자 메서드를 써야 할까?

- 아래의 클래스는 단순히 값만 가지고 있는 클래스이다.
 
- 이러한 클래스는 데이터 필드에 직접 접근할 수 있으니 `캡슐화`의 이점을 제공하지 못한다.

- 그래서 보통 필드의 접근 제한자를 전부 private로 변경 후 getter 메소드를 제공한다.
  
  ```java
  // 옳지 않은 예
  class Point {
    public int x;
    public int y;
  }

  // 옳은 예
  class Point {
    private int x;
    private int y;

    public int getX() {
      return x;
    }

    public int getY() {
      return y;
    }
  }
  ```

  > 📌 **캡슐화의 이점을 제공하지 못한다고 표현한 것에 대한 의견**
  > - 책에서는 캡슐화의 이점을 제공하지 못한다고 헀지만 엄밀히 말해 `정보 은닉`과 `캡슐화`는 다르다.

  > - 캡슐화 (encapsulation)는 객체 안에 속성과 연산을 묶어 넣는 것이다.

  > - 정보 은닉 (information hiding)은 데이터 접근을 제한하여 외부에서는 알것만 알자는 개념이다.

  > - 따라서 아이템 16에서는 `캡슐화`의 이점을 제공하지 못한다는 표현보다 `정보 은닉`의 이점을 제공하지 못한다는 표현이 더 적절하다고 생각된다.


[정보 은닉과 캡슐화의 차이 정리 블로그](https://medium.com/@corebeau87/%EC%BA%A1%EC%8A%90%ED%99%94%EC%99%80-%EC%9D%80%EB%8B%89%ED%99%94%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90-427e5149768d)

<br>

## 접근자 메서드를 사용했을 때의 이점

- 패키지 바깥에서 접근할 수 있는 public 클래스라면 접근자 메서드를 제공함으로써 클래스 내부 표현방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

  ```java
  // 만약 x 필드의 필드명을 변경하고 싶다면?

  // x의 필드명 변경 시 API 스펙이 변경되는 것이므로
  // 클라이언트 코드도 수정되어야 한다.
  class Point {
    public int x;
  }

  // x의 필드명 변경되지만 API 스펙이 변경되는 것은 아니므로
  // 클라이언트 코드는 수정될 필요가 없다.
  class Point {
    private int x;

    public int getX() {
      // x의 변수명을 변경 후, 리턴만 잘 해주면
      // getX()라는 메소드를 변경하지 않고도
      // 내부표현 방식을 변경할 수 있다.
      return x;
    }
  }
  ```

<br>

## 만약 public 클래스가 아닌 private 클래스나 private 중첩 클래스인 경우에는?

- 이 경우는 클래스 자체가 외부로 노출되지 않는 priavte 접근제한자를 가지고 있기 때문에 데이터 필드를 노출해도 (public, default 접근제한자를 사용해도) 문제가 없다.

- 아 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. 왜냐하면 getter 메소드를 쓰면 코드의 길이가 길어지기 때문이다.

  ``` java
  // name 필드에 직접 접근 시
  myclass.name

  // name 필드에 접근자 메서드를 사용할 시
  myclass.getName()
  ```

- 어차피 private 클래스를 사용하는 클라이언트도 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다.
  - 즉 private 클래스를 사용하는 클라이언트도 제한되어 있기 때문에 API 스펙을 변경하는 범위가 적다는 의미이다.

<br>

## 자바에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있다.

- 예를 들어 java.awt 패키지의 Point 클래스와 Dimension 클래스다.

- 자바 17에서도 여전히 똑같이 public 필드를 가지고 있다.
  
<br> 

## 만약 public 클래스의 필드가 불변이라면?

- 필드가 불변이라면 필드를 직접 노출할 떄의 단점이 조금 줄어들지만 여전히 좋은 생각은 아니다.
  - 직접 노출하여도 값을 변경할 수 없다는 점에서 단점이 줄어들었다고 표현한 것 같다.

- 그러나 API 스펙을 변경하지 않고는 필드의 표현 방식을 변경할 수 없고 (필드명을 변경하는 등), 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점이 여전하다.
  - 만약 필드를 읽을 때마다 값을 콘솔에 출력하고 싶다면?
  
  - 만약 필드를 쓸 때 특정 값에 대한 유효성을 체크하고 싶다면?

- 불변 필드를 노출한 public 클래스는 과연 좋을까?
  
  - 불변 필드를 노출했더라도 필드를 읽을 때 부수 작업을 수행할 수 없다는 점은 여전하다.
  
  ```java
  public final class Time {
      private static final int HOURS_PER_DAY = 24;
      private static final int MINUTES_PER_HOUR = 60;

      // 불변 필드를 선언하고 외부로 노출한 경우
      public final int hour;
      public final int minute;

      public Time(int hour, int minute) {
          if (hour < 0 || hour >= HOURS_PER_DAY) {
              throw new IllegalArgumentException(" 시간 : " + hour);
          }
          if (minute < 0 || minute >= MINUTES_PER_HOUR) {
              throw new IllegalArgumentException(" 분 : " + minute);
          }
          this.hour = hour;
          this.minute = minute;
      }
  }
  ```