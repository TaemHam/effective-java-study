# 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

## 다음은 흔히 볼 수 있는 메서드이다.

```java
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 	단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmtpy() ?
    null : new ArrayList<>(cheesesInStock);
}
```

- 재고가 없는 경우 null을 반환한다면, 메서드를 사용하는 클라이언트에서 반드시 null 처리를 해줘야 한다.

- 재고가 비었을 때는 null이 아닌 비어있는 리스트를 반환하면 클라이언트에서 별도의 null 처리를 할 필요가 없다.

- 따라서 null을 반환하기 보단 아래처럼 비어있는 컬렉션을 반환하도록 하자.

<br>

```java
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheeses() {
  return new ArrayList<>(cheesesInStock);
}
```

<br>

## null이 아닌 빈 컬렉션을 반환하면 성능 저하가 있으니 null을 반환하는게 나을까?

- 빈 컬렉션을 할당하는 것이 비용이 들긴 하지만 성능 저하의 주범이 아닌 이상 null 처리를 하는 것보다 낫다.

  - 왜냐하면 클라이언트에서 null 처리를 할 필요도 없고 작성하는 코드도 단순해지기 때문이다.

<br>

## 만약 빈 컬렉션을 반환하는 코드가 성능저하를 일으킨다면?

- 매번 똑같은 불변 컬렉션을 반환하면 된다.

- 자바에서는 Collections 클래스가 비어있는 정적 불변 컬렉션을 제공한다.

  ```java
  public static void main(String... args) {
    Collections.emptyList();
    Collections.emptySet();
    Collections.emptyMap();
  }

  public static final List EMPTY_LIST = new EmptyList<>();
  public static final Set EMPTY_SET = new EmptySet<>();
  public static final Map EMPTY_MAP = new EmptyMap<>();
  ```

- 따라서 비어있는 불변 컬렉션을 반환하는 코드는 다음과 같이 작성된다.

  ```java
  public List<Cheese> getCheeses() {
    return cheesesInStock.isEmtpy()
      ? Collections.emptyList()
      : new ArrayList<>(cheesesInStock);
  }
  ```

<br>

## 배열을 반환할 때도 null이 아닌 빈 배열을 반환하도록 하자.

- 빈 배열을 반환하는 올바른 방법이다.

  ```java
  public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
  }
  ```

- 만약 빈 배열을 반환할 때 성능 저하가 일어난다면 미리 비어있는 빈 배열을 선언해두는 방법을 사용하여 다음과 같이 해결할 수 있다.

  ```java
  private static final Cheese[] EMTPY_CHEESE_ARRAY = new Cheese[0];

  public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMTPY_CHEESE_ARRAY);
  }
  ```

- 그러나 비어있는 배열을 미리 만들어두는 것은 오히려 성능 저하를 일으키는 경우도 있으므로 반드시 테스트를 해야한다.

> **📌 그렇다면 무조건 빈 컬렉션이나 배열을 반환하는게 좋을까?**<br>
>
> - 무조건은 없다! 오히려 정책에 따라 null을 반환하는게 나을 수도 있으니 반드시 빈 컬렉션이나 배열을 반환해야 된다는 건 아니니, 상황에 따라 맞추어 개발하자!
