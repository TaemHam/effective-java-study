# 아이템 31. 한정적 화일드카드를 사용해 API 유연성을 높이라

### 타입 매개변수와 와일드카드, 어느걸 쓰는 게 좋을까?

타입 매개변수와 와일드카드에는 공통된 부분이 있어, 둘 중 어느 것을 사용해도 괜찮을 때가 있다. 예를 들자면, 다음과 같이 메서드의 구현 코드에 제네릭 타입이 등장하지도 않는 상황이 있다.

```JAVA
public static <E> void printCollection(Collection<E> c) {
    for (Object e : c) {
        System.out.println(e);
    }
}
```

이 때, 타입 매개변수 \<E> 를 쓰는게 나을까? 와일드카드 \<?> 를 쓰는 게 나을까?
정답은 '와일드카드를 쓰는게 낫다'이다. 와일드카드는 제네릭이랑 달리 공변이므로, 와일드카드를 쓰면 메서드 구현에서 타입 매개변수를 신경쓸 필요가 없어지기 때문이다.

둘 중 어느 것을 사용해도 좋은지에 대한 판단은 다음의 간단한 규칙만 보면 된다.

> 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.

위의 `public static <E> void printCollection(Collection<E> c)` 에서는 Collection\<E>로 한 번만 등장했기 때문에 다음과 같이 와일드카드로 대체하면 된다.

```JAVA
public static <?> void printCollection(Collection<?> c) {
    for (Object e : c) {
        System.out.println(e);
    }
}
```

하지만 위 규칙을 무조건 따를 수 있는 건 아니다. 다음 예제를 보자.

```JAVA
// i 번째 요소와 j 번째 요소의 자리를 바꿔준다.
public static <?> void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

위 코드를 컴파일 하려고 하면, 다음과 같은 이상한 오류를 내뿜는다:
```
incompatible types: Object cannot be converted to CAP#1
```
원인은 List<?>에 null 외 어떤 타입도 집어넣을 수 없어 일어나는 현상이다. 이 때에는 똑같은 메서드인데 타입 매개변수를 \<E>로 사용하는 제네릭 메서드를 만들어 동작을 위임하는 방법이 있다.

```JAVA
public static <?> void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));    
}
```