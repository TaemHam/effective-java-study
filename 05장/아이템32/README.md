# 아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

### 가변인수란?

가변인수(varargs)란, 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 하는 것이다.

```JAVA
static int sum(int... numbers){
    int sum = 0;
    for (int number : numbers) {
        sum += number;
    }
    return sum;
}
```

## 가변인수와 제네릭

자바 라이브러리에는 제네릭과 가변인수를 사용한 메서드가 여럿 존재한다.

`Arrays.asList(T... a)`, `Collections.addAll(Collection<? super T> c, T... elements)`, `EnumSet.of(E first, E... rest)` 등등

그만큼 제네릭 가변인수 메서드는 실무에서 매우 유용하다.

하지만 이번 아이템의 제목처럼, '제네릭과 가변인수를 함께 쓸 때는 신중'해야 할 만큼 조심해야 하는 상황도 있다. 정확히 어떤 상황을 조심해야 하는지 알아보자.

1. 가변인수로 만들어진 제네릭 배열에 값을 저장하는 경우

가변인수 메서드를 호출하면, 가변인수를 담기 위한 배열이 자동으로 만들어진다.

따라서 제네릭과 가변인수를 함께 사용하게 되면 제네릭 배열이 생성되고,

"아이템 28. 배열보다는 리스트를 사용하라" 에서 설명했던 공변 특성으로 인한 힙 오염 발생 가능성 탓에 아래와 같은 컴파일 경고를 내뱉게 된다.

> warning : [unchecked] Possible heap pollution from
> parameterized vararg type List<String>

조금 억지이긴 하지만, 다음과 같은 코드는 ClassCastException을 발생시킨다.

```JAVA
public class Main {

    static void dangerous(List<String>... stringList) {
        List<Integer> intList = List.of(42);
        Object[] objects = stringList;                   // 공변 특성으로 Object 배열에 담을 수 있고,
        objects[0] = intList;                            // intList로 바뀌고,
        String s = stringList[0].get(0);                 // 여기서 42가 꺼내지며 ClassCastException 발생!
    }

    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");

        dangerous(list);
    }
}
```

역시 아이템 28에서 사용한 예제 그대로이다. get() 으로 나오는 값이 String 이 아니므로, ClassCastException이 발생하게 된다.

이처럼 타입 안전성이 깨지기 때문에, 제네릭 varargs 매개변수에 값을 저장하는 것은 안전하지 않다.

2. 가변인수로 만들어진 제네릭 배열을 클라이언트에게 그대로 노출 시킬 경우

제네릭 varargs 매개변수 배열에 값을 저장하지 않아도, 다른 메서드가 접근하도록 허용하는것 자체만으로도, 의도치 않은 힙 오염이 발생할 수 있으므로 안전하지 않다.

```JAVA
public class PickTwo {

    // 세 개의 인자를 받아, 그 중 두개를 임의로 골라 배열로 반환
    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); 
    }

    // 자신의 제네릭 매개변수 배열의 참조를 그대로 노출
    static <T> T[] toArray(T... args) { 
        return args; // 여기서 args 는 Object[] 배열
    }

    public static void main(String[] args) { 
        String[] attributes = pickTwo("좋은", "빠른", "저렴한"); // ClassCastException 발생
        System.out.println(Arrays.toString(attributes));
    }
}
```

컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열을 만드는 코드를 생성하며, 이 때 배열의 타입은 Object[]이다.

그런데 컴파일러는 pickTwo의 반환값을 attributes에 저장하기 위해 String[]으로 형변환하는 코드를 자동 생성하게 된다.

Object[] 는 String[] 의 하위 타입이 아니므로 이 형변환은 실패하게 되기 때문에, 이 때 ClassCastException 이 발생한다.

### 만약 타입 안정성이 보장된다면 어떻게 해야할까?

아이템 27에서 언급했듯, 비검사 경고는 숨기거나, 그 원인을 제거하거나, 둘 중 한 가지 방법을 택하는 것이 좋다.

```JAVA
// 이 코드를 다음 두 가지 방법에 맞게 고쳐보자.
public class FlattenWithList {
    static <T> List<T> flatten(List<? extends T>... lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }

    public static void main(String[] args) {
        List<Integer> flatList = flatten(
                List.of(1, 2), List.of(3, 4, 5), List.of(6,7));
        System.out.println(flatList);
    }
}
```

1. 경고를 숨기기.

캐스팅이 일어나는 곳에서 `@SuppresssWarnings("unchecked")`을 달아주면 경고가 숨겨진다고 이전 아이템에서 언급한 바 있다.

하지만 이번 경우에는 캐스팅이 클라이언트쪽 코드에서 일어난다. 그럼 호출하는 곳 마다 어노테이션을 붙여야 할까?

자바 7 이후의 버전이라면 @SafeVarargs 을 통해 메서드 작성자가 그 메서드가 타입 안전한지 보장해 경고를 숨길 수 있다.

따라서 힙 오염이 발생하지 않는 등 위 두 상황이 일어나지 않는다면 메서드에 @SafeVarargs 를 달아 쉽게 숨길 수 있다.

```JAVA
@SafeVarargs // 이거 하나 붙여주면 끝   
static <T> List<T> flatten(List<? extends T>... lists) {
    ...
}
```

2. 경고를 제거하기.

만약 @SafeVarargs를 붙일 수 없는 상황이라면, 경고를 제거하는 방법도 있다.

제네릭 가변인수를 사용하는 상황에서 경고 발생 원인은 제네릭과 배열의 혼용에 있다. 즉, 둘 중 하나를 버리고 다른 하나만 택한다면 경고를 제거할 수 있다.

가장 쉬운 방법은 가변인수를 받는 대신 List 매개변수를 받으면 된다.

구현 코드에는 `...`을 없애는 대신 `List<>`로 감싸고, 클라이언트는 `List.of()`로 감싸 인자들을 전달하면 된다.

```JAVA
public class FlattenWithList {
    // 리스트로 받는다.
    static <T> List<T> flatten(List<List<? extends T>> lists) {
    	...
    }

    public static void main(String[] args) {
        // 클라이언트도 List.of() 로 감싸줘야 하는 번거로움은 있다.
        List<Integer> flatList = flatten(List.of(
                List.of(1, 2), List.of(3, 4, 5), List.of(6,7)));
        System.out.println(flatList);
    }
}
```

이 방법은 클라이언트 코드가 살짝 지저분해지고 속도가 조금 느려진다는 단점은 있다.

반대로, 오히려 배열 없이 제네릭만 사용하기 때문에 컴파일러가 이 메서드의 타입 안전성을 검증할 수 있어 더욱 안전하다는 장점도 있다.
