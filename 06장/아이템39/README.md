# 아이템 39. 명명 패턴보다 애너테이션을 사용하라

## 명명 패턴이란?

- 전통적으로 도구나 프레임워크가 특별히 다뤄야할 프로그램 요소에는 명명 패턴을 이용하였다.

- 명명 패턴이란 `변수나 함수의 이름을 일관된 방식으로 작성하는 패턴`을 의미한다.

  - 예를 들어 Junit4 이전에는 `테스트 메서드 이름을 test로 시작`한다는 명명 패턴을 적용하여 테스트를 위한 메서드임을 표시하도록 했었다.

### 명명 패턴의 단점

1. 명명 패턴을 사용할 때 오타가 나면 안 된다.

   - 예를 들어 테스트를 하고자 하는 메서드의 이름이 `test`로 시작해야 테스트 메서드라고 판별할 수 있는데 `tset`으로 오타가 나면 의도하지 않았어도 테스트 메서드로 활용할 수 없다.
   - 명명 패턴을 지키지 않았을 경우 컴파일 오류가 발생하지 않아 불편하다.

2. 올바른 프로그램 요소 (적용하고자 하는 프로그램 요소)에만 명명 패턴이 사용되리라는 보장이 없다.

   - 예를 들어 `테스트 메서드 이름을 test로 시작`한다는 명명 패턴을 사용할 때, 클래스 이름에 `test`를 붙였다고 하자.
   - 개발자는 `test`라고 시작하는 클래스에 작성된 모든 메서드를 junit이 테스트 해 주길 기대할 수도 있지만 의도한 대로 테스트는 진행되지 않으며 경고나 오류도 발생시키지 않는다.

3. 프로그램 요소를 매개변수로 전달할 방법이 없다.
   - 명명 패턴을 사용하면 원하는 매개변수 (예외 타입, 테스트에 필요한 상수 등)들을 전달할 수 있는 방법이 없다.

<br>

## 애너테이션이란?

- 애너테이션이란 메타 데이터를 표현하는 태그이다.

  [애너테이션이란?](https://www.javatpoint.com/java-annotation)

### 명명 패턴의 단점을 해결하는 애너테이션

1. 명명 패턴을 사용할 때 오타가 나면 안 된다.

   - 애너테이션은 별도의 파일로 저장하기 때문에 오타가 나면 컴파일러가 컴파일 오류를 발생시킨다.

2. 올바른 프로그램 요소 (적용하고자 하는 프로그램 요소)에만 명명 패턴이 사용되리라는 보장이 없다.

   - 애너테이션에 @Target이라는 메타 애너테이션을 적용하여 원하는 프로그램 요소에만 애너테이션을 적용할 수 있다.

3. 프로그램 요소를 매개변수로 전달할 방법이 없다.
   - 애너테이션은 원하는 타입의 매개변수를 전달할 수 있는 방법을 제공한다.

<br>

## 애너테이션 사용하기 1 - 기본적인 사용방법

```java
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 이 애너테이션은 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {

}
```

- @Retention, @Target 애너테이션은 애너테이션 선언 시 추가적인 정보를 제공하는 메타 애너테이션이다.

  > **📌 메타 애너테이션이란?**<br>
  >
  > - 메타 애너테이션은 다른 애너테이션 선언에 다는 애너테이션을 의미한다.

  - @Retention은 선언한 애너테이션이 어느 시점에 적용될지를 알려주는 애너테이션이다.

  - @Target은 선언한 애너테이션이 어느 요소에 적용될지를 알려주는 애너테이션이다.

- 위의 애너테이션의 주석에는 `이 애너테이션은 매개변수 없는 정적 메서드 전용이다.`라는 제약이 있는데 이를 컴파일러가 강제할 수 있게 하여면 javax.annotation.processiong API를 사용하여 애너테이션 처리기를 구현해야 한다.

```java
public class Sample {
    @Test
    public static void m1() {
        // @Test가 적용되었으므로 성공해야 한다.
    }

    public static void m2() {
        // @Test가 적용되지 않았으므로 테스트 대상 메서드가 아니다.
    }

    @Test
    public static void m3() {
        // 테스트는 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m4() {
        // @Test가 적용되지 않았으므로 테스트 대상 메서드가 아니다.
    }

    @Test
    public void m5() {
        // @Test는 적용되었으나 static 메서드가 아니므로 테스트 대상 메서드가 아니다.
    }

    public static void m6() {
        // @Test가 적용되지 않았으므로 테스트 대상 메서드가 아니다.
    }

    @Test
    public static void m7() {
        // 테스트는 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m8() {
        // @Test가 적용되지 않았으므로 테스트 대상 메서드가 아니다.
    }
}
```

```java
// Sample 클래스의 메서드 중 @Test가 적용된 메서드만 체크한다.
public class Main {

    public static void main(String[] args) throws ClassNotFoundException {
        int tests = 0;
        int passed = 0;

        Class<?> testClass = Class.forName("org.example.mypackage.annotation.Sample");

        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    // 정적 메서드를 호출할 때는 1번째 파라미터를 null로 부여하면 된다.
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + "실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test : " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}


// 결과는 다음과 같이 출력된다.
// 성공: 1, 실패: 3
```

<br>

## 애너테이션 사용하기 2 - 매개변수 사용하기

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

```java
public class Sample2 {

    @ExceptionTest(ArithmeticException.class)
    public static void m1() {
        // 테스트는 성공해야 한다.
        int i = 0;
        i = i / i;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() {
        // 명시한 예외가 아닌 다른 예외가 발생하므로 테스트는 실패해야 한다.
        int[] a = new int[0];
        int i = a[1];
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m3() {
        // 예외가 발생하지 않으므로 테스트는 실패해야 한다.
    }
}
```

```java
// Sample2 클래스의 메서드 중 @ExceptionTest가 적용된 메서드만 체크한다.
public class Main {

    public static void main(String[] args) throws ClassNotFoundException {
        int tests = 0;
        int passed = 0;

        Class<?> testClass = Class.forName("org.example.mypackage.annotation.Sample2");

        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    // 정적 메서드를 호출할 때는 1번째 파라미터를 null로 부여하면 된다.
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    Class<? extends Throwable> excType =
                            m.getAnnotation(ExceptionTest.class).value();

                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest : " + m);
                }
            }
        }
    }
}
```

<br>

## 애너테이션 사용하기 3 - 매개변수 배열 사용하기

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    // 변수 타입이 배열로 변경되었다.
    Class<? extends Throwable>[] value();
}

```

```java
public class Sample3 {

    @ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
    public static void doublyBad() {
        // 테스트는 성공해야 한다.
        List<String> list = new ArrayList<>();

        // 5번째 인덱스의 요소에 null을 넣는 메소드
        // 비어있는 리스트이므로 IndexOutOfBoundsException가 발생하며
        // 동시에 NullPointerException도 발생한다.
        list.addAll(5, null);
    }
}
```

```java
// Sample3 클래스의 메서드 중 @ExceptionTest가 적용된 메서드만 체크한다.
public class Main {

    public static void main(String[] args) throws ClassNotFoundException {
        int tests = 0;
        int passed = 0;

        Class<?> testClass = Class.forName("org.example.mypackage.annotation.Sample3");

        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    Class<? extends Throwable>[] excTypes =
                            m.getAnnotation(ExceptionTest.class).value();

                    // 배열 매개변수를 처리하는 로직
                    for (Class<? extends Throwable> excType : excTypes) {
                        if (excType.isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed) {
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                    }
                }
            }
        }
    }
}
```

<br>

## 애너테이션 사용하기 4 - 반복 가능한 애너테이션 사용하기

- 애너테이션에 여러 개의 매개변수를 전달할 때 배열 매개변수 대신 @Repeatable 메타 애너테이션을 달아서 처리하는 방법도 있다.

- @Repeatable 메타 애너테이션을 단 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다.

- 사용방법은 다음과 같다.
  - 실제로 사용할 애너테이션에 @Repeatable 애너테이션을 달아준다.
  - @Repeatable 애너테이션에 `컨테이너 애너테이션`을 선언 후 매개변수로 넣어준다.

```java
// 실제로 사용할 애너테이션을 선언한다.
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

// 컨테이너 애너테이션을 선언한다.
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    // 애너테이션 자체를 담는 배열 변수를 선언한다.
    ExceptionTest[] value();
}

```

```java
// ExceptionTest를 2번 작성하여 여러 개의 매개변수를 전달한다.
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
    // 테스트는 성공해야 한다.
    List<String> list = new ArrayList<>();

    // 5번째 인덱스의 요소에 null을 넣는 메소드
    // 비어있는 리스트이므로 IndexOutOfBoundsException가 발생하며
    // 동시에 NullPointerException도 발생한다.
    list.addAll(5, null);
}

```

### 반복 가능한 애너테이션 사용 시 주의 사항

- 반복 가능한 애너테이션을 여러 개 달면 하나만 달았을 때와 달리 `컨테이너 애너테이션` 타입이 적용된다.

  - 이 예제에서 컨테이너 애너테이션은 @ExceptionTestContainer이다.

- 만약 다음 메서드를 사용한다면 위의 예제에서 입력한 2개의 예외가 전부 배열에 담긴다.

  ```java
  // IndexOutOfBoundsException, NullPointerException 2개가 담긴 배열이 반환된다.
  ExceptionTest[] annotationsByType = m.getAnnotationsByType(ExceptionTest.class);
  ```

- 그러나 isAnnotationPresent 메서드는 반복 가능 애너테이션과 컨테이너 애너테이션을 명확히 구분한다.

  - 반복 가능 애너테이션이 1개일 때는 그 애너테이션이 있다고 판정한다.
  - 반복 가능 애너테이션이 여러 개일 때는 컨테이너 애너테이션이 있다고 판정한다.

    ```java
    // @ExceptionTest를 여러 개 달았을 때는 false, 1개 달았을 때는 true 반환
    System.out.println(m.isAnnotationPresent(ExceptionTest.class));

    // @ExceptionTest를 여러 개 달았을 때는 true, 1개 달았을 때는 false 반환
    System.out.println(m.isAnnotationPresent(ExceptionTestContainer.class));
    ```

```java
public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        int tests = 0;
        int passed = 0;

        Class<?> testClass = Class.forName("org.example.mypackage.annotation.Sample3");

        for (Method m : testClass.getDeclaredMethods()) {
            // ExceptionTest가 1개일 때와 여러 개인 경우
            // true를 반환하는 파라미터 조건이 다르기 때문에
            // 아래와 같이 2개의 조건 모두 체크를 해 줘야 한다.
            if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed) {
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                    }
                }
            }
        }
    }
}

```

<br>

## 애너테이션을 쓰자!

- 메서드 이름, 클래스 이름 등의 명명 패턴을 통한 규칙보다 애너테이션을 사용한 규칙이 훨씬 낫다.

- 왜냐하면 컴파일 오류를 사용할 수 있으며, 문자열이 아닌 코드로써 제약을 걸 수 있는 점이 좋다.

- 애너테이션을 사용하면 보일러 플레이트 코드 등을 간단하게 처리할 수 있다. (롬복 등)

- 단, 애너테이션을 사용하면 작성해야할 코드의 양이 늘어나므로 반드시 오류가 발생할 가능성이 커짐을 알고 있어야 한다.

<br>

## 추가 학습

### 애너테이션 처리기 사용하기

- 애너테이션을 작성하고, AbstractProcessor 클래스를 상속받아 애너테이션 처리기를 만들 수 있다.

- 단, 애너테이션 처리기를 작성한 애너테이션을 사용하기 위해서는 별도의 모듈로 선언해야 하며 아래 링크를 참고하면 좋다.

  [애너테이션과 애너테이션 처리기](https://stackoverflow.com/questions/55715819/integrate-annotation-processor-into-the-same-project)

<br>

### 스프링 AOP와 애너테이션

- 스프링 AOP의 포인트 컷을 작성할 때 애너테이션을 사용할 수 있다.

```java
// 포인트 컷을 애너테이션을 사용하여 만들기
@Around("@annotation(hello.aop.member.annotation.MethodAop)")
```

```java
// 애너테이션을 사용한 어드바이스 만들기

@Around("@annotation(hello.aop.member.annotation.MethodAop)")
public Object processCustomAnnotation(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
    // AOP 처리 로직 작성..
}
```
