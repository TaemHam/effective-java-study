# 아이템 45. 스트림은 주의해서 사용하라

## 스트림

스트림이란 자바 8부터 다량의 데이터 처리 작업(순차/병렬)을 돕고자 나온 API로, 원하는 결과를 생성하기 위해 `파이프라인`으로 연결될 수 있는 다양한 메서드를 지원하는 개체 시퀀스이다. 데이터의 흐름, 즉 스트림이 흘러가는 과정에서 데이터가 순차적으로 하나씩 사용되고 사라진다.

대용량 데이터를 처리할 때 효율을 높이기 위해, 오토박싱/언박싱 과정이 필요 없는 Intstream 과 같은 기본형 스트림도 제공한다.

* 스트림의 추상 개념
1. 스트림(stream) : 데이터 원소의 유한 혹은 무한 시퀀스
2. 스트림 파이프라인(stream pipeline) : 이 원소들로 수행하는 연산 단계를 표현

## 스트림 파이프라인

스트림 파이프라인은 소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다.

![파이프라인](https://velog.velcdn.com/images/semi-cloud/post/7f73e3a6-941c-4c4d-af7e-b3a19d1b921e/image.png)

```JAVA
List<Integer> transactionsIds = 
    transactions.stream()
                .filter(t -> t.getType() == Transaction.GROCERY)
                .sorted(comparing(Transaction::getValue).reversed())
                .map(Transaction::getId)
                .collect(toList());
```

1. 중간 연산(Intermediate Operation)<br>
    각 중간 연산은 스트림을 어떠한 방식으로 변환하는 역할을 수행하며, 모두 한 스트림을 다른 스트림으로 변환하게 하여 메서드 체이닝을 가능하게 한다. 예를 들어, 각 원소에 함수를 적용하거나 특정 조건을 만족 못하는 원소를 걸러낼 수 있다.

    `.filter()`, `.map()`, `.flatmap()`, `.sorted()`

2. 종단 연산(Terminal Operation)<br>
    종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가하는 역할을 한다. 예를 들어, 원소를 정렬해 컬렉션에 담거나 특정 원소를 하나 선택하는 식이다.

    `.forEach()`, `.collect()`, `.match()`, `.count()`, `.reduce()`

### 스트림 파이프라인 특징

1. 지연 평가(lazy evaluation)

    먼저 지연이란 결과값이 필요할때까지 계산을 늦추는 기법을 의미한다. 이렇게 함으로써 어느 부분에서 가장 큰 이익을 얻을 수 있을까?

    대용량의 데이터에서, 실제로 필요하지 않은 데이터들을 탐색하는 것을 방지해 속도를 높일 수 있다. 즉, 종단 연산에 쓰이지 않는 데이터 원소는 계산 자체에 쓰이지 않는다. 그리고 이것을 `Short-Circuit` 방식이라 부른다.

    `.limit()`, `.findFirst()`, `.findAny()`, `.anyMatch()`, `.allMatch()`, `noneMatch()`

    스트림 파이프라인을 실행하게 되면, JVM 은 곧바로 스트림 연산을 실행시키지 않는다. 최소한으로 필수적인 작업만 수행하고자 검사를 먼저 하고, 이를 바탕으로 최적화 방법을 찾아내 계획한다. 그리고 그 계획에 따라 개별 요소에 대한 스트림 연산을 수행한다.

    예를 들어, 10000 개의 데이터중에 길이가 5가 넘는 문자열에서 가장 알파벳순으로 앞에 있는 2개의 문자열만 가지고 오고 싶다고 하자. 지연 평가가 없이 순서대로 바로 동작했다면, 10000 개의 데이터를 모두 순회해야 했을 것이다.

2. 순차성

    기본적으로 스트림 파이프라인은 순차적으로 수행된다. 파이프라인을 병렬로 실행하려면, 파이프라인을 구성하는 스트림 중 하나에서 parallel 메서드를 호출해주기만 하면 되나, 효과를 볼 수 있는 상황은 많지 않다.

### 스트림이 무조건 옳을까?

스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수가 힘들어진다.

```JAVA
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Interger.parseInt(args[1]);
        Map<String, Set<String>> groups = new HashMap<>();

        try(Scanner s = new Scanner(dictionary)) {
            while(s.hasNext()) {
                String word = s.next();
                // 맵 안에 키가 있다면 매핑된 값을 반환하고 
                // 없다면 건네진 함수 객체를 키에 적용하여 값을 계산한 다음
                // 키와 값을 매핑해놓고 계산된 값을 반환
                groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
            }
        }
        
        for(Set<String> group : groups.values())
            if(group.size() >= minGroupSize)
                System.out.println(group.size() + ":" + group);
    }
    
    public static String alphabetize(String s){
    	char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

위 클래스는 사전 파일에서 단어를 읽어 사용자가 지정한 문턱값보다 원소 수가 많은 아나그램 그룹을 출력한다. (철자를 구성하는 알파벳이 같고 순서만 다른 단어) 이를 위해 아나그램끼리는 같은 키를 공유하도록 구현되었다.

그리고 아래는 스트림을 과하게 사용해 리팩토링한 코드이다.

```JAVA
    public static void main(String[] args) throws IOException {
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                        StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
```

확실히 짧긴 해도 읽기가 어렵다.

이렇게 과하게 사용하지 않고, 적절하게 사용하는 방법도 있는데, 그 방법은 다음과 같다.

```JAVA

    public static void main(String[] args) throws IOException {
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    public static String alphabetize(String s){
    	char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
```

여기서 alphabetize 함수도 스트림을 사용해 구현할 수 있는데, 그렇게 하지 않는 편이 좋다.<br>
명확성이 떨어지고, 느려질 수도 있고, 잘못 구현할 수 있기 때문이다.<br>
이 이유는 자바가 char용 스트림을 지원하지 않기 때문이다.

잘못 구현할 수 있다는 건, 아래와 같은 상황이 발생하기 때문이다.
``` JAVA
"Hello World!".chars().forEach(System.out::print);
```
위 코드의 결과는 739488237102... 으로, 각 char을 int 값으로 변환해 출력해버린다.

**결론적으로, 기존 코드는 스트림을 사용하도록 리팩터링 하되, 새 코드가 나을 때에만 반영하는게 좋다.**

### 스트림의 적절한 활용

스트림 파이프라인은 되풀이 되는 계산을 주로 함수 객체(람다/메서드 참조)로 표현하고, 반복 코드에는 코드 블록을 사용해 표현한다.

**스트림으로 처리하기 힘든 경우**

스트림과 함수 객체로는 할 수 없지만, 코드 블록으로는 할 수 있는 경우를 구분하면 다음과 같다.

1. 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 사실상 final 인 변수만 읽을 수 있고, 지역변수 수정은 불가능하다.

2. 코드 블록에서는 return / break / continue 문으로 블록의 흐름을 제어하거나, 메서드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다는 불가능하다.

3. 코드 블록에서는 이전 값을 변수에 저장해 접근할 수 있다. 하지만 스트림은 데이터가 파이프라인을 통과하면 이전의 값을 잃기 때문에, 앞에서 사용한 데이터를 뒤에서 접근해야 하는 경우는 스트림을 쓰기 어렵기도 하고 어찌저찌 사용해도 보기 지저분해진다.

**스트림을 적용하기 좋은 경우**

1. 원소들의 시퀀스를 일관되게 변환해야 하는 경우 : map()

2. 원소들의 시퀀스를 필터링 해야 하는 경우 : filter()

3. 원소들의 시퀀스를 하나의 연산을 사용해 결합해야 하는 경우(더하기, 연결하기, 최솟값 구하기 등) : reduce()

4. 원소들의 시퀀스를 컬렉션에 모으는 경우(공통된 속성을 기준으로) : collect()

5. 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾을 경우 : filter()