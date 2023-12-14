# 아이템 61. 박싱된 기본 타입보다는 기본 타입을 사용하라

## 자바의 데이터 타입은 크게 2가지로 나뉜다.

- 첫 번째 타입으로는 short, int, long, float, double, boolean, byte, char인 기본 데이터 타입이 있다.

- 두 번째로는 String, List와 같은 참조형 타입이 있다.

- 자바에서는 기본 타입에 대응하는 참조 타입이 하나씩 있다.

  - ex. Integer, Long, Boolean 등

  - 이를 박싱된 기본 타입이라고 한다. (또는 Wrapper 클래스라고도 함)

<br>

## 오토 박싱, 오토 언박싱

- 자바에서는 오토 박싱, 오토 언박싱을 지원하므로 기본 타입과 박싱된 기본 타입 간 타입 캐스팅을 명시하지 않아도 된다.

- 그러나 타입 캐스팅을 명시하지 않아도 되는 것이지, 내부적으로는 타입 캐스팅 연산이 이루어지기 때문에 기본 타입과 박싱된 기본 타입의 차이를 아는 것이 중요하다.

<br>

## 기본 타입과 박싱된 기본 타입의 차이

- 첫 번째 차이점

  - 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 `참조형 타입`이기 때문에 식별성이란 속성을 갖는다.
  - 즉 인스턴스 주소가 다르면 다른 객체라고 인식할 수 있다는 것이다.

- 두 번째 차이점

  - 기본 타입은 항상 유효한 값을 가지나, 박싱된 기본 타입은 null을 가질 수 있다.

- 세 번째 차이점
  - 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

<br>

## Comparator를 통해 오토박싱으로 인한 오류를 알아보자

```java
Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```

- 위의 Comparator에서는 `박싱된 정수 값`을 받아 비교하는 비교자이다.
  - 이제 선언한 naturalOrder를 사용해서 Integer를 비교해보자

```java
static Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);

public static void main(String[] args) {
    Integer integer = Integer.valueOf(500);
    // new Integer(42)를 쓰면 deprecated 되었으므로 컴파일 오류가 발생한다.
    // 대신 Integer.valueOf를 쓰면 된다.
    int compare = naturalOrder.compare(Integer.valueOf(500), Integer.valueOf(500));
    System.out.println(compare);
}
```

- 이 때 `naturalOrder.compare(Integer.valueOf(500), Integer.valueOf(500));`의 결과가 같음을 의미하는 0이 나오길 원하지만 1이 나오는 것을 확인할 수 있다.

- 이는 두 객체의 값은 같더라도 인스턴스는 다르기 때문에 다른 객체라고 판단하는 것이다.

  - String을 비교할 때 == 를 쓰지 않는 것과 같다.

- 그러므로 박싱된 기본 타입을 비교할 때는 주의해야하며, 이 경우에는 Comparator.naturalOrder()를 쓰자.

- 만약 굳이 비교자를 만들어야 한다면 박싱된 기본 타입 간 비교가 아닌 언박싱 후 기본 타입을 비교하는 것이 좋다.

  > **📌 참고**<br>
  >
  > - 책에서 소개한 new Integer()는 deprecated 되어서 컴파일 오류가 발생하므로 사용할 수 없다.
  > - 따라서 Integer.valueOf()를 사용해서 테스트하였다.
  > - 책의 예제와 같이 42로 하면 Integer.valueOf(42)와 Integer.valueOf(42)를 비교하면 비교 결과가 0이 나온다.
  > - 그러나 Integer.valueOf(500)과 Integer.valueOf(500)을 비교하면 비교 결과가 1이 나온다.
  > - 그 이유는 Integer.valueOf 메서드에서 -128 ~ +127 까지의 값은 캐싱 후 캐싱된 결과를 반환하기 때문이다.

<br>

## 오토 박싱을 혼용해서 쓰는데는 주의가 필요하다.

```java
static Integer i;

public static void main(String[] args) {
    if (i == 42) {
        System.out.println("NPE 발생!");
    }
}
```

- 위의 예제를 보자. 박싱된 기본 타입인 Integer i를 활용하여 값을 비교하는 로직이다.

- 위의 코드를 실행하면 NPE가 발생한다.

- 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입이 자동으로 풀린다. (오토 언박싱)
  - 박싱된 기본 타입은 null 까지 표현할 수 있는데, null일 때 오토 언박싱이 되면 NPE가 발생 해버린다.

<br>

```java
public static void main(String[] args) {
    Long sum = 0L;

    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }

    System.out.println(sum);
}
```

- 위의 예제에서는 for 루프가 실행될 때마다 sum을 언박싱 후 값을 더하고 다시 박싱하므로 성능이 굉장히 느려진다.

<br>

## 그렇다면 박싱된 기본 타입은 언제 써야 할까?

- 첫 번째는 컬렉션의 원소, 키, 값으로 쓴다. 즉, 제네릭으로 값을 받는 타입에 기본 타입과 같은 값을 넣어주고 싶다면 박싱된 기본 타입을 써야 한다.

- 리플렉션을 쓸 때도 박싱된 기본 타입을 사용해야 한다. (문법 상 써야한다.)
