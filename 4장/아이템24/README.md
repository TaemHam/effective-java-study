# 아이템 24. 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스 (nested class)의 종류

### 정적 멤버 클래스

- 정적 멤버 클래스는 다른 클래스 안에 선언된다.
 
- 정적 멤버 클래스는 바깥 클래스의 private 필드에 접근할 수 있다.

    ```JAVA
    public class OuterClass{

        private String name;

        static class NestedClass {
            // NestedClass를 감싸고 있는 바깥 클래스인 OuterClass의
            // private 필드에도 접근할 수 있다.
        }
    }
    ```

- 정적 멤버 클래스는 바깥 클래스와 함께 쓰이는 public 도우미 클래스로 쓰인다.

    ```JAVA
    public class Calculator{
        // 바깥 클래스와 함께 쓰일 경우이므로, 논리적으로 연관이 있어야 한다.
        public static class Operator {
            static String PLUS;
            static String MINUS;
        }
    }

    public void static main(String... args) {
        // Calculator와 연관된 Operator에 접근하여 사용한다.
        String plus = Calculator.Operator.PLUS;
    }


    ```

- 정적 멤버 클래스의 예시
  
    ```java
    public class InnerClasses {

        static class MyStaticInnerClass {
            static void print() {
                System.out.println("정적 멤버 클래스에서 호출함!");
            }
        }
    ```

    <br>

    ```java
    public static void main(String[] args) {

        // 정적 멤버 클래스에 접근하기
        InnerClasses.MyStaticInnerClass.print();
    }
    ```

<br>

### 비 정적 멤버 클래스

- 정적 멤버 클래스와 비 정적 멤버 클래스의 차이는 static 키워드의 유무이지만 의미상으로는 차이가 크다.

- 비 정적 멤버 클래스의 인스턴스는 `바깥 클래스의 인스턴스` 의 인스턴스와 암묵적으로 연결된다.

- 따라서 바깥 클래스의 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다.
    - 왜냐하면 비 정적 멤버 클래스는 바깥 클래스를 인스턴스로 만든 후에야 인스턴스로 만들 수 있기 때문이다.

- 보통 어댑터 패턴에 많이 쓰인다. (기능을 사용할 클래스를 중첩 클래스로 선언하고 부가 기능을 나의 클래스에 선언하는 경우가 있다.)

<br>

```java
public class NestedNonStaticExample {

    private final String name;

    public NestedNonStaticExample(String name) {
        this.name = name;
    }

    public String getName() {
        // 비정적 멤버 클래스와 바깥 클래스의 관계가 확립되는 부분
        NonStaticClass nonStaticClass = new NonStaticClass("nonStatic : ");
        return nonStaticClass.getNameWithOuter();
    }

    // 비 정적 메서드 시작
    private class NonStaticClass {
        private final String nonStaticName;

        public NonStaticClass(String nonStaticName) {
            this.nonStaticName = nonStaticName;
        }

        public String getNameWithOuter() {
        // 정규화된 this 를 이용해서 바깥 클래스의 인스턴스 메서드를 사용할 수 있다.
            return nonStaticName + NestedNonStaticExample.this.getName();
        }
    }
}
```

<br>

- 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 (독립적으로 쓰인다면) 무조건 static을 붙여서 정적 멤버 클래스로 만들자.
  - static으로 만들어서 메모리 누수를 막을 수 있다. (static이 아니면 내부 클래스의 인스턴스가 필요없더라도 바깥 클래스의 인스턴스가 GC에 의해 해제되지 않는 이상 계속 메모리에 남아있다.)

> **📌 참고**<br><br>
> 자바의 Map.Entry도 static은 아니지만 interface로 선언하여 static으로 만들었다.
> 따라서 Entry는 메모리 누수의 우려가 없다.

- 비 정적 멤버 클래스의 예시
  
    ```java
    public class InnerClasses {

        class MyInnerClass {
            void print() {
                System.out.println("비 정적 멤버 클래스에서 호출함!");
            }
        }
    }
    ```

    <br>

    ```java
    public static void main(String[] args) {

        // 멤버 클래스에 접근하기
        InnerClasses.MyInnerClass myInnerClass = new InnerClasses().new MyInnerClass();
    }
    ```

<br>

### 익명 클래스
- 익명 클래스는 쓰이는 시점에 선언과 동시에 사용된다.

- 익명 클래스는 별도의 클래스를 만들지 않더라도 내가 원하는 시점에 원하는 로직을 선언할 수 있기는 한데, 가독성이 떨어지며 여러가지 제약이 많다.
  - 선언한 지점에서만 인스턴스를 만들 수 있다.

- 자바 8 이후에는 익명 클래스는 람다 표현식을 통해 사용할 수 있게 되었으므로 보다 가독성을 높여 사용할 수 있다.
  - 람다 표현식은 익명 클래스를 단순화 한 것이다!

- 익명 클래스의 예시
  
    ```java
    public class InnerClasses {

        interface MyInterface {
            void print();
        }
    }
    ```

    <br>

    ```java
    public static void main(String[] args) {

        // 익명 클래스 사용하기
        // 인터페이스를 바탕으로 익명클래스 사용하기
        InnerClasses.MyInterface anonymousWithInterface = new InnerClasses.MyInterface() {
            @Override
            public void print() {
                System.out.println("익명 클래스에서 치리");
            }
        };

        // 클래스를 바탕으로 익명클래스 사용하기
        InnerClasses.MyInnerClass anonymousWithClass = new InnerClasses().new MyInnerClass() {
            @Override
            void print() {
                super.print();
            }
        };
    }
    ```
<br>

### 지역 클래스

- 지역 변수를 선언할 수 있는 위치라면 어디든 지역 클래스를 선언할 수 있다.

- 메소드 안, 인스턴스 필드 선언 위치 등등..

- 지역 클래스의 예시

    ```java
    public static void main(String[] args) {

        // 지역 클래스 사용하기 (메서드 안에 클래스를 선언한 예시)
        class LocalClass {
            void print() {
            }
        }

        LocalClass localClass = new LocalClass();
        localClass.print();
    }
    ```