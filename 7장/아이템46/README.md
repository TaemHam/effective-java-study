# 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라

## 스트림 패러다임

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다.
이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 `순수 함수`여야 한다.

> **📌 참고**<br>
>
> 순수 함수란?
>
> 오직 입력만이 결과에 영향을 주는 함수. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않아야 한다.<br>
> 순수 함수는 언제 어디서 호출하든 같은 값을 입력하면 무조건 같은 결과를 도출한다는 특징이 있다.

이렇게 하려면 스트림 연산에 건네는 함수 객체는 모두 **부수 효과(side effect)가 없어야 한다**. 부수 효과란, 자신의 스코프 밖 객체의 상태를 참조하거나 변경함으로 함수를 호출했을 때 다른 결과가 반환되는 현상을 의미한다.

### 순수 함수 예제

다음은 어떤 text 파일에 있는 단어들의 등장 횟수를 세어 빈도표로 만드는 코드이다.

```JAVA
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

위 코드는 다음의 이유로 스트림 API의 이점을 살리지 못한 스트림 코드를 가장한 반복적 코드라고 할 수 있다.
1. forEach 라는 의도가 명확하지 않은 연산을 사용했다. collect 의 경우 "원소를 돌면서 특정한 컬렉션으로 모은다" 라는 의도가, reduce 라면 "원소를 돌면서 하나로 뭉친다" 라는 의도가 표현되는데 반해, forEach는 "원소를 돌면서 무언가를 한다"밖에 표현할 수 없다.
2. 모든 작업이 forEach 에서 일어나는데, 이 때 외부 상태(freq)를 수정하는 람다를 실행하고 있다. 이렇게 되면, 이 함수는 해당 외부 상태에 종속적이게 되므로, 외부 상태에 따라 다른 결과를 반환할 수 있다.

위 코드를 개선한다면 아래와 같이 바꿀 수 있다.

```JAVA
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

결론적으로 forEach 연산은 종단 연산 중 기능이 가장 적고 병렬화할 수도 없어서 덜 "스트림" 답기 때문에, 계산 결과를 보고할 때만 사용하고 계산하는 데는 쓰지 말아야 한다.

> **📌 참고**<br>
> forEach 연산은 병렬화 할 수 없다는 건 무슨 뜻일까?
>
> 스트림을 열 때 `.parallelStream()`으로 열면 별도의 코어에서 병렬로 파이프라인을 진행시킬 수 있다.<br> 
>이 때, 병렬 스트림의 실행 순서는 개발자가 통제할 수 없는데, 이를 forEach() 를 사용해 결과를 종합한다면 순서가 엉망인 채로 결과가 반환되어버린다.<br>
> 하지만 collect() 나 reduce() 같은 다른 종단 연산을 사용하면, 종단 연산이 적용되기 전에 중간 연산이 적용된 결과를 기다리고, 모두 모이면 종단 연산을 사용해 결과를 반환한다. 즉, 결과 반환의 순서가 보장이 된다. 
>
> 병렬 스트림에 관한 더욱 자세한 내용은 [[여기](https://sas-study.tistory.com/461)]에 더 잘 나와있다.

## Collector (수집기)

앞의 예제로 증명했듯, 스트림의 원소들을 연산 적용 후 한 곳에 모으기 위해 컬렉션 객체를 바깥에 선언하고 원소를 넣어주는 것은 좋지 못하다. 그렇다면 어떻게 컬렉션으로 모을 수 있을까?

정답은 위에서 사용했듯, collect() 종단 연산을 사용하는 것이다.

앞서 말했듯, 스트림의 종단 연산으로 collect() 를 사용하면 원소들을 하나로 뭉쳐 컬렉션으로 반환할 수 있게 된다. 이 collect() 메서드에 무얼 넣어주느냐에 따라 List나 Set, 그리고 다른 컬렉션으로도 바꿀 수 있게 된다. 그리고 이 메서드에 매개변수로 넣어주어야 하는 것이 `Collector` 이다.

이 듣도 보도 못한 Collector를 어떻게 만들어서 넣어주어야 하는지 걱정할 필요는 없다. <br> `java.util.stream.Collectors` 라이브러리가 정적 메서드로 여러가지 방법으로 원소들을 뭉쳐주는 Collector를 39개나 제공해주니, 대부분의 경우는 이 라이브러리를 사용하면 된다.

* Collectors.toList() 와 toSet()

이름만 봐도 알 수 있듯, 각각 List와 Set의 형태로 반환해준다. 구현체는 각각 ArrayList와 HashSet이다.

어떤 IDE는 `.collect(Collectors.toList())`를 `.toList()`로 변환하도록 추천해준다. 더욱 짧고, 의도도 뚜렷하고 해서 추천대로 변환해도 문제가 없어보인다. 하지만 여기에 숨겨져있는 함정은, toList()는 **`UnmodifiableList`**를 반환한다는 것이다. 즉, 결과로 받은 List는 원소를 추가하거나 삭제할 수 없는, 불변 객체가 되는 것이다. 같은 형식의 toSet()도 마찬가지이다. [[참고](https://binux.tistory.com/146)]

* Collectors.toCollection()

ArrayList나 HashSet이 아닌, 다른 컬렉션의 형태로 만들어주고 싶은 경우 사용한다.
어떤 컬렉션으로 만들어주고 싶은지는 매개변수로 넣어주는데, 이 때 넣어주는 매개변수는 실제 인스턴스가 아닌, 인스턴스를 만들어주는 팩터리를 넣어주어야 한다. 예를 들어, `TreeSet`으로 만들고 싶다면, new TreeSet<>() 이 아닌, 람다식인 `() -> new TreeSet<>()` 나 메서드 참조형인 `TreeSet::new`를 넣어주는 형식이다.

* Collectors.toMap()

역시 HashMap을 구현체로 가지는 Map 형태로 바꿔주는 컬렉터이다. 위의 toList()와 다르게, 이 메서드는 안에 필수적으로 2개 + 추가적으로 2개의 매개변수 넣어주어야 한다. 세 개의 매개변수는 각각 다음과 같다.

필수: 

1. 원소를 받고, 키로 넣어줄 값을 반환하는 Function
2. 원소를 받고, 밸류로 넣어줄 값을 반환하는 Function

옵션:

3. 같은 키를 가진 두 밸류를 받아, 키 충돌난 밸류를 처리한 값을 반환하는 BiFunction<br> (a, b를 받아 a를 반환하면 이전 값으로 설정, b를 반환하면 새로운 값으로 설정, max(a, b)면 둘 중 최댓값으로 설정 등), <br> 설정 안하면 키 충돌시 예외 반환 
4. toCollection 처럼, 기본 설정된 HashMap 이 아닌 다른 Map 구현체를 사용하고 싶을 때 설정

예제 코드는 아이템 34에서 보았던 계산기 Enum을 가져왔다.

```JAVA
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {return x + y;}
    },
    MINUS("-") {
        public double apply(double x, double y) {return x - y;}
    },
    TIMES("*") {
        public double apply(double x, double y) {return x * y;}
    },
    DIVIDE("/") {
        public double apply(double x, double y) {return x / y;}
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public String getSymbol() {
        return symbol;
    }

    public abstract double apply(double x, double y);

    // symbol을 키로, 각 열거형을 밸류로 맵에 저장
    private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(toMap(Operation::getSymbol, e -> e));

    // Optional을 반환해 클라이언트에게 null 값이 나올수 있음을 알린다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
}
```

여기서 toMap은 1번과 2번 매개변수만 넣어줬다. 맵은 "+"를 키로 Operation.Plus 가 저장된 형태를 가진다.

* Collectors.groupingBy()

위의 toMap()과 비슷하지만, 키 값 충돌이 나면 둘 중 하나만 저장하는 게 아닌, 같은 키 값을 가지는 밸류 모두 한 컬렉션 안에 모아주는 컬렉터다. 즉, 밸류로는 Set 이든 List든 무조건 컬렉션을 가지게 된다.

매개변수로 필수적으로 1개, 추가적으로 2개를 받는다.

필수:

1. 원소를 받고, 키로 넣어줄 값을 반환하는 Function

옵션:

2. toCollection 처럼, 기본 설정된 HashMap 이 아닌 다른 Map 구현체를 사용하고 싶을 때 설정
3. 같은 키 값을 가지는 밸류를 어떤 컬렉션으로 저장할 지 설정하는 컬렉터<br>
(toList() 나 toSet(), 심지어는 또 다시 groupingBy()를 넣어 맵 안에 맵 안에 컬렉션을 넣을 수도 있다)

아래의 예제는 아이템 37에서 가져온 것으로, 식물을 생명주기별로 묶어 Set으로 저장하는 코드이다.

```JAVA
public class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }

    public static Map<LifeCycle, Set<Plant>> classifyByLifeCycle(List<Plant> plants) {
        return plants.stream()
                .collect(groupingBy(
                        plant -> plant.lifeCycle,             // 무얼 기준으로 분류할 건지 (키)
                        () -> new EnumMap<>(LifeCycle.class), // 무슨 자료형으로 저장할 건지 (맵의 자료형)
                        toSet()));                            // 분류하고 난 밸류는 어떻게 저장할 건지 (밸류의 컬렉터)
    }

    public static void main(String[] args) {
        List<Plant> plants = List.of(
                new Plant("Asters", LifeCycle.PERENNIAL),
                new Plant("Beets", LifeCycle.BIENNIAL),
                new Plant("Cosmos", LifeCycle.ANNUAL)
        );
        Map<LifeCycle, Set<Plant>> plantsByLifeCycle = Plant.classifyByLifeCycle(plants);
        // 결과 출력
        System.out.println(plantsByLifeCycle);
    }
}
```

이 외에도, 

* 뭉치고 나서 또 다른 작업을 수행하는 `.collectingAndThen()`, <br>
* String 스트림을 하나의 String으로 뭉치는 `.joining()`, <br>
* 어떤 함수를 기준으로 true, false로 나눠 컬렉션으로 저장하는 `.partitioningBy()`, <br>
* 요소들의 통계 수치(갯수, 평균, 합계, 최대, 최소)를 내는 `.summarizingDouble/Long/Int()`, <br>
* 위의 통계 수치 중 하나를 반환하는 `.counting()`, `.averagingDouble/Long/Int()`, `.summingDouble/Long/Int()`, `.maxBy()`, `.minBy()`

가 있다.

자세한 용법은 [[여기](https://www.daddyprogrammer.org/post/1163/java-collectors/)]를 참조하길 바란다.