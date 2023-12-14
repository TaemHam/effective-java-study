# 아이템 52. 다중정의는 신중히 사용하라

재정의(Overriding)와 다중정의(Overloading)은 이름도 비슷하고, 하는 일도 이름이 같은 다른 메서드를 호출하는 것으로 매우 비슷해 보이지만, 둘은 완전히 다른 일을 한다고 할 수 있을정도로 다르다.

오늘은 이 두 유사한 키워드로 인해 자주 발생하는 오류와 그 오류를 피하기 위해선 어떻게 하는지를 다루어 볼 것이다.


## 재정의와 다중정의

### 재정의(Overriding)

재정의란, **상위 클래스가 정의한 것과 같은 시그니처의 메서드를 하위 클래스에서 다시 정의한 것**을 의미한다.

메서드를 재정의한 다음 '하위 클래스 인스턴스'에서 그 메서드를 호출하면 재정의한 메서드가 실행되며, 컴파일 타임의 인스턴스 타입은 신경쓰지 않는다.

```Java
class Wine {
    String name() { return "포도주"; }
}
class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}
class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}
public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList)
            System.out.println(wine.name());  
            // "포도주", "발포성 포도주", "샴페인"
    }
}
```

위 코드를 보면, `wineList`에 담긴 객체들은 `List<Wine>` 에 담겨있으므로, 컴파일 타임에는 `Wine` 타입이다. 하지만 for 문에서 각 객체를 꺼내며 `name()` 메서드를 실행하면, 컴파일 타입과 무관하게 항상 '가장 하위에서 정의한' 재정의 메서드가 실행된다.

이는 사실 '재정의'라는 맥락에서, 그 의미대로 동작한 것이니 보통 문제가 되지는 않는다.

### 다중정의(Overloading)

하지만 다중정의는 다르다.

```Java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
            // "그 외", "그 외", "그 외"
    }
}
```

위 코드는 `collections` 안의 객체가 Set인지, List인지, 아니면 그 외의 객체인지 따라 다른 메서드를 호출하도록 의도한 클래스 분류기이다. `collections` 안의 객체가 순서대로 [Set의 한 종류, List의 한 종류, 그 외의 종류] 이기 때문에, "집합", "리스트", "그 외" 가 출력될 것이라고 예상한다. 하지만 애석하게도 실제 결과는 "그 외" 만 3번 출력되어버린다.

이런 오류가 발생하는 이유는, 다중 정의의 경우 **객체가 호출할 메서드가 컴파일타임에 정해지기 때문**이다.  즉, for 문 안의 c는 항상 Collection<?> 타입이기 때문에 `classify(Collection<?> c)` 만 호출되는 것이다.

올바르게 사용하려면 다음과 같이 classify 메서드를 하나로 합친 후 instanceof로 명시적으로 검사하면 말끔히 해결된다.

``` Java
public class FixedCollectionClassifier {
    public static String classify(Collection<?> c) {
        return c instanceof Set  ? "집합" :
                c instanceof List ? "리스트" : "그 외";
    }
    ...
}
```

### 다중정의 올바르게 사용하기

다중정의를 오류 없이 사용하려면 다음의 규칙을 따르면 된다.

1. 매개변수의 수가 같은 다중정의는 만들지 말자.
2. 가변인수(varargs)를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다.

만약, 위의 규칙을 따르기 힘든 상황이라면, 메서드의 이름을 다르게 지어주는 걸 고려해보는 것도 좋다. ObjectOutputStream이 기본 타입을 사용하는 비슷한 메서드를 `writeBoolean(boolean), writeInt(int), writeLong(long)`으로 정해준 것 처럼 말이다.

### 다중정의 주의사항

1. 생성자

생성자는 이름을 다르게 지을 수 없으니, 두 번째 생성자부터는 무조건 다중정의가 되어버린다. 따라서 생성자도 위에서 언급한 규칙을 따르거나, 혹은 생성자에 이름을 붙여줄 수 있는 정적 팩터리 메서드를 사용할 수 있을 것이다.

[[정적 팩터리 메서드](https://github.com/TaemHam/effective-java-study/tree/main/2%EC%9E%A5/%EC%95%84%EC%9D%B4%ED%85%9C01)]

2. 오토박싱

매개변수 수가 같은 다중정의 메서드가 많더라도, 매개변수중 하나 이상이 "근본적으로 다르다"면, 즉 서로 형변환 될 수 없다면 앞에서 설명한 오류가 발생하진 않을 것이다. 어떤 다중정의 메서드를 호출할지가 매개변수들의 컴파일이 아닌 **런타임 타입**만으로 결정되기 때문이다.

하지만 오토박싱이 도입되고, 기본 타입과 참조 타입이 근본적으로 다름을 보장할 수 없게 되면서 문제가 발생했다.

```JAVA
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list); 
        // [-3, -2, -1] [-2, 0, 2] 
    }
}
```

이 프로그램은 -3부터 2까지의 정수를 TreeSet과 ArrayList에 더하고, 다시 두 컬렉션에 0부터 2까지 remove()를 호출하는 프로그램이다.

우리는 이 프로그램이 두 컬렉션에서 동일하게 작동하도록 기대하겠지만, 결과는 그렇지 않다.

`Collection.remove(Object)`는 해당 Object와 같은 값을 가지는 원소를 제거하는 기능을 가졌기에, `Set.remove(1)`은 1의 값을 가진 원소를 제거해준다.

하지만 `List.remove()`는 `remove(Object)`와 `remove(int)`로 다중정의가 되어있기에, 그리고 `remove(int)`는 해당 정수를 인덱스로 가지는 원소을 제거하는 기능을 하기에, `List.remove(1)`은 1의 인덱스를 가진 원소를 제거하며 다른 결과를 반환하는 것이다.

### 람다, 메서드 참조

다중정의 메서드들(혹은 생성자)들이 함수형 인터페이스를 인수로 받을 때, 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다. 달라 보이는 함수형 인터페이스도 근본적으로 다르지 않기 때문이다.

```JAVA
// 1. Thread의 생성자 호출
new Thread(System.out::println).start();

// 2. ExecutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println); // 컴파일 에러
```

위의 1번 코드와 2번 코드 모두 Runnable을 받는 메서드를 다중정의하고 있는데, 2번 코드만 컴파일 오류를 내뿜는다. 

원인은 바로 `exec.submit()`이 Callable을 받도록 다중정의가 되어 있고, `System.out::println` 또한 Runnable을 반환하거나 Callable을 반환하도록 다중정의가 되어 있어, 둘 중 어느 쪽을 사용하는지 판단할 수 없기 때문이다.