# 아이템 55. 옵셔널 반환은 신중히하라

## 자바 8 이전에 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 방법

- null을 반환하기

  - 그러나 null을 반환하면 클라이언트가 null 처리를 해줘야 하는 단점이 있다.

- 예외를 발생시키기
  - 예외는 진짜 예외인 상황에서만 사용해야 하며 예외를 생성할 때 스택 추적 전체를 캡처하므로 그에 따른 비용도 적지 않게 든다.

<br>

## 자바 8부터 도입된 선택지

### Optional을 사용하자

> **📌 Optional이란?**<br>
>
> - Optional은 값이 있는지 없는지를 표현하는 컨테이너 오브젝트를 의미한다.
> - Optional은 원소를 최대 1개 가질 수 있는 불변 컬렉션이다.
>   - Optional을 할당하면 그 Optional의 값을 변경할 수는 없다.

- Optional을 반환하면 null을 반환하는 것보다 오류 가능성이 적고, 예외를 던지는 메서드보다 유연하고 사용하기 쉽다.

- 다음은 Optional을 사용하는 예제이다.

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if (c.isEmpty()) {
    // 컬렉션이 비어있으면 비어있는 Optional을 반환한다.
    return Optional.empty();
  }

  E result = null;

  for (E e : c) {
    if(result == null || e.compareTo(result) > 0) {
      result = Objects.requireNonNull(e);
    }
  }

  return Optional.of(result);
}
```

> **📌 주의!**<br>
>
> - Optional을 반환하는 메서드에는 null을 반환하게 하지 말자.
> - 왜냐하면 Optional을 사용하는 메서드에서 null을 반환하는 건 Optional을 사용하는 취지를 무시하는 것이다.

- 재고가 없는 경우 null을 반환한다면, 메서드를 사용하는 클라이언트에서 반드시 null 처리를 해줘야 한다.

- 재고가 비었을 때는 null이 아닌 비어있는 리스트를 반환하면 클라이언트에서 별도의 null 처리를 할 필요가 없다.

- 따라서 null을 반환하기 보단 아래처럼 비어있는 컬렉션을 반환하도록 하자.

<br>

## Optional 사용하기

- 값이 존재하지 않으면 기본값을 사용하기
  ```java
  optional.orElse("기본값");
  ```
- 값이 존재하지 않으면 예외를 던지기
  ```java
  optional.orElseThrow(() -> new IllegalArgumentException("유효하지 않은 값입니다."));
  ```
- 항상 값이 존재한다고 가정하기
  ```java
  optional.get();
  ```
- 값이 존재하는지 판별하기

  ```java
  optional.isPresent();
  ```

  <br>

### Optional의 API는 static으로 제공하므로 stream에서도 편리하게 사용할 수 있다.

- Stream<Optional\<T>> 을 사용하여 Optional에서 값을 추출하기

  ```java
  optional
    .filter(Optional::isPresent)
    .map(Optional::get);

  ```

- Optional에 stream 사용하기

  - Optional에 stream을 사용하면 optional이 담고 있는 값을 stream으로 반환한다.

    > **📌 주의!**<br>
    >
    > - Optional에 List\<Integer>가 들어있을 때 Optional.stream의 반환값은 Stream\<Integer>가 아닌 `Stream<List<Integer>>`임에 주의하자.
    > - 아래에 나오지만, 컬렉션을 Optional로 감싸는 것은 좋은 방법이 아니긴 하다.

    <br>

    ```java
    streamOptionals
      .flatMap(Optional::stream)
    ```

    <br>

## Optional을 무조건 사용한다고 해서 이득은 아니다!

- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 Optional로 감싸지 말자. (그렇다고 무조건적인 것은 아니다!)

  - 비어있는 Optional<List\<T>> 를 반환하는 것이 아닌 비어있는 리스트를 반환하도록 하자.

- Optional을 사용하는 것은 Optional이라는 컨테이너 객체를 사용하는 것이므로 그에 따른 비용이 발생한다.

  - 기본 타입을 Optional에 사용하는 것은 2중으로 감싸는 것이 된다.

  - 그러므로 OptionalInt, OptionalLong, OptionalDouble 같은 클래스를 사용하자.

- 값의 존재성을 나타낼 때 무조건 Optional을 사용하는 것이 정답은 아니다.

  - 예를 들어 Map<Integer, Optional\<T>>을 사용한다고 하면 특정 key에 대해 값이 없다는 표현을 이중으로 사용하는 것이 된다.

  - map.get(key) == null, map.get(key).isPresent() 두 가지 방법으로 표현할 수 있는 것이다.

  - 이는 혼란만 가중 시키며 쓸데없이 복잡한 코드가 되므로 주의하자.

<br>

## Optional은 언제 쓸까?

- 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 하는 메서드라면 Optional을 반환해야할 수 있는 상황일 수 있다.

  - 대표적으로 JpaRepository가 있다.

- 그러나 성능이 민감하다면 값이 없음을 표현할 때 Optional을 사용하는 것이 아닌 null이나 예외를 던지는 것이 나을 수 있다.
