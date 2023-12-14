# 아이템 28. 배열보다는 리스트를 사용하라

### 배열과 리스트는 무슨 차이가 있는데?

1. 배열은 공변(Covarient)이고, 제네릭은 불공변이다.

-   공변 = Sub와 Super의 하위 타입이라면, 배열 Sub[]도 배열 Super[] 의 하위 타입이 된다.

2. 배열은 실체화(reify)되고, 제네릭은 그렇지 않다.

배열은 런타임에도 타입 검사를 진행하지만, 제네릭은 타입 정보가 컴파일 후 소거된다. 제네릭 도입 전의 코드와 호환성을 보장해주기 위함.

### 이 차이가 어떤 상황을 만드는데?

1. 배열은 런타임 오류를 만들어 디버깅이 힘들게 만든다.

공변 특성을 가지는 배열은 상위 타입 배열 변수에도 담기기 때문에, 다음 상황에 런타임 오류를 뿜는다.

```Java
public static void main(String[] args) {
        String[] names = new String[5];
        doSomethingWithArray(names);
    }

private static void doSomethingWithArray(Object[] objects) {
    ...

    objects[0] = 1; // 런타임에 ArrayStoreException 발생!
}
```

불공변 특성을 가지는 제네릭으로 위의 코드를 재현하면, 컴파일 자체가 되지 않는다.

```Java
public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        doSomethingWithArray(names); // 여기서 컴파일 오류 발생!
    }

private static void doSomethingWithArray(List<Object> objects) {
    ...
}
```

2. 제네릭으로 배열을 생성할 수 없다.

런타임에 타입이 소거되는 제네릭을 런타임에 타입을 확인해야하는 배열안에 넣을 수 없다.

`new List<E>[]`, `new List<String>[]`, `new E[]` 식으로 작성하면 컴파일 오류를 뿜는다.

List<E>[] 나 E[] 는 그렇다 쳐도, List<String>[] 은 왜 안될까?

이유는 타입 안전하지 않기 때문이다.

> 타입 안전이란, 언어가 타입 오류를 얼마나 잘 억제하거나 방지하는지를 설명하는 특성

```JAVA
public static void main(String[] args) {
    List<String>[] stringList = new List<String>[1]; // 만약 이걸 허용한다고 하면,
    List<Integer> intList = List.of(42);             //
    Object[] objects = stringList;                   // 공변 특성으로 Object 배열에 담을 수 있고,
    objects[0] = intList;                            // intList로 바뀌고,
    String s = stringList[0].get(0);                 // 여기서 42가 꺼내지며 컴파일 오류가 날 것!
}
```

### 그럼 배열을 제네릭 컬렉션으로 어떻게 바꿀까?

특정 원소 집합 중 랜덤으로 원소를 뽑아주는 choose() 메서드를 가지는 Chooser 클래스를 보자.

```JAVA
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }

    public Object choose() {
        Random random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)];
    }
}
```

위 클래스는 choose 메서드로 뽑은 랜덤 원소를 내가 원하는 타입으로 형변환 해줘야하는 번거로움이 있다.

이제, 이 클래스를 제네릭으로 만들어보자.

먼저, Obejct를 T로 바꿔보자.

```Java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray(); // 여기서 컴파일 오류가 날 것이다.
    }

    public T choose() {
        Random random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)];
    }
}
```

이렇게 바꾸면 동작할 것 같지만, Object[] 배열이 T[] 배열로 다운 캐스팅이 안되었으니 컴파일 오류를 뿜을 것이다.

그럼 이 `Object[]` 배열을 `T[]` 배열로 형변환 하면 될까?

`choiceArray = (T[]) choices.toArray();`

이러면 컴파일은 가능하지만, unchecked 경고 메시지가 뜬다. T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없기 때문이다.

만약 이 형변환이 런타임에도 안전하다고 확신한다면, 주석을 남기고 @SuppressWarning을 달아 경고를 숨기는 방법도 있다.

하지만 애초에 배열을 사용하지 않고 리스트를 사용하도록 바꾼다면, 형변환이 필요 없으니 경고도 뜨지 않게 된다.

다음이 배열 대신 리스트를 사용한 최종 버전이다.

```Java
public class Chooser<T> {
    private final List<T> choiceArray;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random random = ThreadLocalRandom.current();
        return choiceList.get(random.nextInt(choiceArray.size()));
    }
}
```

이 버전은 코드양이 조금 늘었고 조금 더 느릴 수 있지만, 런타임에 ClassCastException이 발생할 수 없으니 그 가치가 있다고 할 수 있다.