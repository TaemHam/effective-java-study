# 아이템 44. 표준 함수형 인터페이스를 사용하라

## 표준 함수형 인터페이스란?

- 표준 함수형 인터페이스란 자바에서 표준으로 제공하는 함수형 인터페이스를 말한다.

- java.util.function 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨있기 때문에 필요한 용도에 맞는게 있다면 직접 구현하기보다는 표준 함수형 인터페이스를 활용하는 것이 좋다.

- java.util.function에는 총 43개의 인터페이스가 있지만 기본 인터페이스 6개에서 확장된 인터페이스들이 많으므로 걱정하지 말자.

<br>

## 기본 인터페이스

> **📌 자바에서 제공하는 기본 함수형 인터페이스**<br><br>
> |인터페이스|함수 시그니처|예시|설명|
> |:-:|:-:|-|-|
> |UnaryOperator<T>|(단항 연산자) T → T|String::toLowerCase|T를 받아 T를 반환한다. (Fuction으로 처리할 수 있지만 더 간단하게 가능한 모양)
> |BinaryOperator<T>|(T, T) → T|BigInteger::add|T 2개를 받아 T를 반환한다. (2개를 합쳐서 1개 만드는 모양)
> |Predicate<T>|T → boolean|Collection::isEmpty|1개의 인수를 받아 boolean을 반환한다.
> |Function<T, R>|T → R|Arrays::asList|T를 인수로 받아 R로 변환한다.
> |Supplier<T>|() → T|Instant::now|아무것도 안 받고 T를 반환한다. (T를 공급한다.)
> |Consumer<T>|T → void|System.out::println|1개의 인수를 받아 void를 반환한다. (요소를 처리하고 끝)

<br>

## 기본 인터페이스를 확장한 인터페이스

- 기본 인터페이스를 바탕으로 `int, long, double`을 사용하여 각 3개씩 변형이 생긴다.

- `IntPredicate` → int를 받는 Predicate는 IntPredicate가 되며, 정수 1개를 받아 boolean을 반환하는 역할을 수행하는 람다 함수를 할당할 수 있다.

- `LongBinaryOperator` → long을 받아 long을 반환하는 역할을 수행하는 람다 함수를 할당할 수 있다.

- `LongFunction<int[]>` → long을 받아 int[]을 반환하는 역할을 수행하는 람다 함수를 할당할 수 있다.

- `BooleanSupplier` → boolean을 반환하는 역할을 수행하는 람다 함수를 할당할 수 있다.

<br>

## 기본 함수형 인터페이스와 다르게 매개변수를 2개씩 받는 인터페이스

- 기본적으로 `Bi` 접두사가 붙는다.

- `BiPredicate<T, U>` → 2개의 파라미터를 받아 boolean을 반환하는 역할을 수행하는 람다 함수를 할당할 수 있다.

- `BiConsumer<T, U>` → 2개의 파라미터를 받아 void를 반환하는 역할을 수행하는 람다 함수를 할당할 수 있다.

- `ToIntBiFunction<T, U>` → 2개의 파라미터를 받아 int를 반환하는 역할을 수행하는 람다 함수를 할당할 수 있다.

  > 📌 **참고**
  >
  > - 자바에서 다양한 함수형 기본 인터페이스의 변형을 제공하고 있으니 기본 인터페이스에 기본 타입을 넣어서 사용하지는 말자.
  > - 예를 들어 `Predicate<Integer>` 보다 `IntPredicate`를 사용하는 것이 좋다.

<br>

## 함수형 인터페이스는 언제 만드는 것이 좋을까?

- 예를 들어 매개 변수를 3개 이상 받아야 하는 Predicate 같이 표준 함수형 인터페이스로 제공되지 않는 것은 직접 구현해야 한다.

- 표준 함수형 인터페이스와 구조가 같더라도 다음과 같은 이유로 직접 작성해야 하는 때도 있다.

  - 자주 쓰이며, 이름 자체가 용도를 명확히 설명해 줄 때

    - Comparator 인터페이스는 이름 자체로도 용도를 명확히 설명해준다.

  - 반드시 따라야 하는 규약이 있을 때

    - Comparator 인터페이스는 두 개의 객체를 비교할 때 쓰이는 명확한 규약인 equals를 구현체에서 지키도록 하고 있다.

  - 유용한 디폴트 메서드를 제공할 수 있을 때
    - Comparator 인터페이스는 다양한 디폴트 메서드를 제공한다.

- 직접 만든 함수형 인터페이스에는 반드시 `@FunctionalInterface` 어노테이션을 붙여주자.

  - @FunctionalInterface는 오직 1개의 추상 메서드만 가질 수 있도록 제약을 걸어준다.
