# 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

열거형 타입 원소들의 순서를 알려주는 `ordinal()`을 각 원소의 정숫값에 대입해 사용하지 말라는 것은 아이템 35에서 언급했다.

그렇다면 정숫값으로 사용하지 않을 때에는 괜찮을까?

다음 코드는 식물을 생애주기별로 저장하기 위해 열거형 타입을 사용한 Plant 클래스이다.

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
```

이 때, 식물들을 배열로 관리해 생애주기별로 묶어주는 코드를 `ordinal()`로 작성해보자.

```JAVA
    // 식물을 생애주기별로 분류
    // 책과는 다르게 List로 만들어보았다.
    public static List<Set<Plant>> classifyByLifeCycle(List<Plant> plants) {
        List<Set<Plant>> plantsByLifeCycle = new ArrayList<>();
        for (int i = 0; i < LifeCycle.values().length; i++) {
            plantsByLifeCycle.add( new HashSet<>());
        }
        for (Plant plant : plants) {
            plantsByLifeCycle.get(plant.lifeCycle.ordinal()).add(plant);
        }
        return plantsByLifeCycle;
    }
```

Plants를 가진 myGarden을 생애주기별로 묶어 출력을 해보자.  

```JAVA
    // 실제로 동작시켜보고 싶으면 이 코드까지 전부 합쳐보자.
    public static void main(String[] args) {
        List<Plant> plants = List.of(
                new Plant("Asters", LifeCycle.PERENNIAL),
                new Plant("Beets", LifeCycle.BIENNIAL),
                new Plant("Cosmos", LifeCycle.ANNUAL)
        );
        List<Set<Plant>> plantsByLifeCycle = Plant.classifyByLifeCycle(plants);
        // 결과 출력
        for (int i = 0; i < plantsByLifeCycle.size(); i++) {
            System.out.printf("%s : %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle.get(i));
        }
    }
}
```

동작은 하지만 문제가 많다.

- **리스트의 인덱스에는 의미가 담겨있지 않다**. 따라서 리스트를 받아 사용하는 클라이언트는 0번 인덱스에 담긴 것은 뭔지, 1번 인덱스에 담긴 것은 뭔지 등 그 의미를 모를 수 있다. 만약 안다 해도, 사용 시에 직접 레이블을 달아야 하는 번거로움이 있다.

- 분류는 열거형으로 했을지언정, 저장 장소는 인덱스로, 즉 정수로 분류되어있기 때문에, **정수 열거 패턴처럼 타입 안정성이 떨어진다**. 위의 경우로 보면, 분류 가짓수가 똑같이 3개인 다른 열거형이 추가됐을 때 (원뿌리, 잔뿌리, 수염뿌리) 둘 중 어떤 분류 기준으로 나뉜 리스트인지 헷갈릴 수 있다. 

위 문제들은 배열이나 리스트를 사용하는 대신, EnumMap을 사용하는 것으로 모두 해결 가능하다.

## EnumMap

EnumMap은 인스턴스 생성시 넣어준 열거 타입을 키로 값을 저장하는 아주 빠른 Map 구현체이다.

위의 코드를 수정한다면 다음처럼 바꿀 수 있다.

```JAVA
    public static Map<LifeCycle, Set<Plant>> classifyByLifeCycle(List<Plant> plants) {
        Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(LifeCycle.class);
        Arrays.stream(LifeCycle.values())
                .forEach(lifeCycle -> plantsByLifeCycle.put(lifeCycle, new HashSet<>()));
        plants.forEach(plant -> plantsByLifeCycle.get(plant.lifeCycle).add(plant));
        return plantsByLifeCycle;
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

EnumMap을 사용함으로 다음 장점을 얻을 수 있다.

- 레이블이 이미 전부 달려있다. 클라이언트 입장에서 사용하기 편하다.
- 타입 안정성도 뛰어나다. 사용한 Enum이 키로 저장되어있기 때문이다.
- 배열과 비교해도 성능이 비등하다. EnumMap은 내부에서 배열을 사용하기 때문이다.

위의 `classifyByLifeCycle` 코드가 너무 번잡하다면 좀 더 깔끔한 방법도 존재한다.

```JAVA
    public static Map<LifeCycle, Set<Plant>> classifyByLifeCycle(List<Plant> plants) {
        return plants.stream()
                .collect(groupingBy(
                        plant -> plant.lifeCycle,             // 무얼 기준으로 분류할 건지 (키)
                        () -> new EnumMap<>(LifeCycle.class), // 무슨 자료형으로 분류할 건지 (키 타입)
                        toSet()));                            // 분류하고 난 밸류는 어떻게 저장할 건지 (밸류 컬렉션 종류)
    }
```

위 코드는 조금 더 깔끔하긴 하지만, 정확히 말하자면 이전의 코드와 구현 방식이 조금 다르다. <br>
plants 리스트 안에  `new Plant("Beets", LifeCycle.BIENNIAL)` 가 없이, 두 종류의 식물만 있다고 해보자. <br>
이전 코드는 BIENNIAL을 키로 가진 빈 Set도 만들지만, <br>
아래 코드는 BIENNIAL이 존재하지 않는다.

ordinal 대신 EnumMap을 사용할 수 있는 경우가 하나 더 있다.

바로 같은 열거형 안에 존재하는 타입간의 관계를 정의하는 경우다.

다음은 물질의 상태인 고체, 액체, 기체와 각 상태 사이에 일어나는 상태 변화를 ordinal을 이용해 정의한 코드이다.

```JAVA
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
       
	    // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.   
        private static final Transition[][] TRANSITIONS = {
            {null, MELT, SUBLIME},
            {FREEZE, null, BOIL},
            {DEPOSIT, CONDENSE, null}
        };
        
        // 한 상태에서 다른 상태로의 상태 변화를 반환한다.
        // 고체(SOLID), 액체(LIQUID)를 넣으면 액화(MELT)가 반환된다.
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

깔끔해 보이지만, 다음과 같은 단점을 가진다.

- Phase에 정의된 상태들이 바뀌면(순서 수정 등), TRANSITIONS 도 똑같이 바뀌어야 한다. 위의 경우엔 SOLID 와 LIQUID의 위치가 개발자의 실수로 바뀌어 버린다면, 고체 -> 액체 = 응고 라는 이상한 결과가 나와버린다.
- 존재하지 않는 관계를 정의하기 위해 null이 쓰이고 있는데, 열거형이 확장된다면 null로 채워지는 공간이 늘어나 공간이 낭비된다. 만약 플라즈마 상태를 추가해 이온화와 탈이온화를 정의하고 싶다면, 행과 열의 추가로 생겨나는 공간 7칸 중 이온화, 탈이온화를 제외한 5칸이 null로 채워지며 공간 낭비가 심해지게 된다.

EnumMap을 사용하도록 바꾼다면, 관계를 2차원으로 표현했듯 EnumMap도 중첩으로 사용하면 된다.

```JAVA
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // "시작 상태 -> 끝 상태 = 상태 변화" 로 저장
        private static final Map<Phase, Map<Phase, Transition>> transitionMap = Stream.of(Transition.values())
                .collect(groupingBy(                       // 바깥쪽 맵 관련
                        transition -> transition.from,     // 키 뭐임?
                        () -> new EnumMap<>(Phase.class),  // 키 타입은 뭐임?
                        toMap(                             // 밸류는 무슨 컬렉션? // 이 밑부터 안쪽 맵 관련
                                transition -> transition.to,         // 키 뭐임?
                                transition -> transition,            // 밸류 뭐임?
                                (oldValue, newValue) -> newValue,    // 같은 키 다른 밸류면 뭘로 저장?
                                () -> new EnumMap<>(Phase.class)))); // 키 타입은 뭐임?

        public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }
}
```

코드는 조금 복잡해졌으나, 위에서 언급한 단점이 모두 보완되었다. 

위에서 언급한 플라즈마 상태와 이온화, 탈이온화가 추가된다면, Phase에 플라즈마를, Transition에 이온화, 탈이온화만 추가하면 된다.

```JAVA
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
        
		... 이하 생략
    }
}
```