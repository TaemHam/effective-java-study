# 아이템 49. 매개변수가 유효한지 검사하라

"오류는 가능한 한 빨리 (발생한 곳에서) 잡아야 한다"는 일반 원칙이 있다. 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워지기 때문이다.

메서드나 생성자의 경우는, 입력 매개변수의 값이 특정 조건을 만족했을 때 제대로 동작해야 한다. 예를 들면, 정수 매개변수가 배열의 인덱스로 사용된다면 음수면 안된다거나 하는 조건이 있다. 이런 매개변수의 조건들은 반드시 문서화해야 하며, 메서드 몸체가 시작되기 전에 검사해야 한다.

## 매개변수 검사

매개변수 검사를 제대로 하지 못하면 문제가 생길 수 있다.
메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있고, 운이 나쁘면 이상한 값을 내보내며 정상작동을 할 수 있고, 운이 더 나쁘면 메서드는 잘 수행되었지만 객체의 값을 바꿔, 미래에 이 메서드와는 관련 없는 오류를 낼 수도 있다.

public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다. 자바독을 이용하면 `@throws` 태그를 사용해 쉽게 문서화할 수 있다.

만약 매개변수의 제약을 문서화한다면, 그 제약을 어겼을 때 발생하는 예외도 함께 기술하는 게 좋다. 

아래는 그 예시다.

```Java
/**
 * (현재 값 mod m) 값을 반환한다. 이 메서드는
 * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 * @param m 계수(양수여야 한다)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0) {
        throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
    }
    ... 계산 수행
}
```
이 메서드에서 m이 null이면 NullPointerException을 던진다는 말은 설명에 없다. 그 이유는 이 설명을 개별 메서드가 아닌 BigInteger 클래스 수준에서 기술했기 때문이다.

클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 훨씬 깔끔한 방법이다.

* null 검사

이전에는 어떤 객체가 null 이 아님을 보장하기 위해서 if 문으로 확인하거나 메서드를 따로 선언해주어야 했다.

하지만 자바 7에 `java.util.Objects.requireNonNull` 메서드가 추가되면서, 더 이상 null 검사를 수동으로 하지 않아도 되게 되었다.

아래는 Objects 클래스의 `requireNonNull` 메서드다.

```Java
public class Objects {
    // ...

    /**
     * Checks that the specified object reference is not {@code null}. This
     * method is designed primarily for doing parameter validation in methods
     * and constructors, as demonstrated below:
     * <blockquote><pre>
     * public Foo(Bar bar) {
     *     this.bar = Objects.requireNonNull(bar);
     * }
     * </pre></blockquote>
     *
     * @param obj the object reference to check for nullity
     * @param <T> the type of the reference
     * @return {@code obj} if not {@code null}
     * @throws NullPointerException if {@code obj} is {@code null}
     */
    public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }

    // ...

    public static <T> T requireNonNull(T obj, String message) {
        if (obj == null)
            throw new NullPointerException(message);
        return obj;
    }
}
```
그리고 위 메서드는 다음과 같이 사용한다.

`this.strategy = Objects.requireNonNull(strategy, "전략");`

정적 퍼블릭 메서드인 `Objects.requireNonNull()` 메서드는, 필요한 곳 어디서든 호출해 null이 아님을 보장할 수 있다. 

* 범위 검사

자바 9에서는 Objects에 범위 검사 기능도 더해졌다.

`checkFromIndexSize()`, `checkFromToIndex()`, `checkIndex()`라는 메서드들이다.

유용하지만 예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계됐다. 또한 닫힌 범위(closed range; 양 끝단 값을 포함하는)는 다루지 못한다.

### 단언문(assert)을 사용한 매개변수 유효성 검증

공개되지 않은 메서드라면 메서드가 호출되는 상황을 작성자 본인이 통제할 수 있다. 즉, 메서드가 받는 매개변수가 유효하다는 것을 단언할 수 있는 상황이며, 때문에 private 메서드는 단언문(assert)을 사용해 매개변수 유효성을 검증할 수 있다.

다음은 재귀 정렬용 함수에서 assert를 사용해 매개변수를 검증하는 메서드이다.

```Java
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... 계산 수행
}
```

이 단언문들은 자신이 단언한 조건이 무조건 참이라고 선언한다. 참이 아닐경우 AssertionError가 발생한다.

단언문은 다음 두 가지 관점에서 일반적인 유효성 검사와 다르다.

1. 실패하면 AssertionError를 던진다.

2. 런타임에 아무런 효과도, 아무런 성능 저하도 없다(단, java를 실행할 때 명령줄에서 -ea 혹은 --enableassertions 플래그 설정하면 런타임에 영향을 준다).

### 나중에 쓰기 위해 저장하는 매개변수의 유효성을 검사하라

메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경써서 검사해야 한다.

오류가 발생한다면, 운이 나쁘면 한참 뒤에서야 문제를 발견할 수도 있다.

생성자는 "나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라"는 원칙의 특수한 사례다. 생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 꼭 필요하다.

### 그러면 매개변수 유효성을 무조건 검사해야 하나?

앞에서 "메서드 몸체 실행 전에 매개변수 유효성을 검사해야 한다"고 말해줬지만 역시 예외는 있다.

1. 유효성 검사 비용이 지나치게 높거나 상용적이지 않을 때
2. 계산 과정에서 암묵적으로 검사가 수행될 때

다만, 암묵적 유효성 검사에 너무 의존하면 실패 원자성을 해칠 수 있으니 주의해야 한다.

> **📌 참고** <br>
> 실패 원자성(Failure-Atomic)이란?
> 
> 호출된 메서드가 실패 하더라도 호출된 메서드의 객체는 메서드 호출 전 상태를 유지해야 한다는 규칙이다.
> 실패 원자적인 메소드는 예외가 발생하더라도, 객체의 일관성이 유지되어 정상적으로 사용할 수 있는 상태를 유지해야 한다.