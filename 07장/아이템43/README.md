# 아이템 43. 람다보다는 메서드 참조를 사용하라

## 메서드 참조란?

- 메서드 참조란 메서드를 파라미터로 전달하여 특정 메서드를 사용하라는 코드를 보다 간결하게 작성할 수 있는 방법을 말한다.

- 예를 들어 다음과 같이 단순화 할 수 있다.

  ```java
  Map<String, Integer> map = new HashMap<>();
  String key = "key";

  // map에 요소를 합치는 merge 메서드를 사용한 에제이다.
  // 만약 map에 key가 존재하지 않는다면 입력받은 key에 value 1을 할당하여 추가한다.
  // 만약 map에 key가 있다면 key에 해당하는 value를 증가시킨다.
  // count는 map에 key에 매핑된 value를 의미하며
  // incr는 merge 메서드의 2번째 파라미터 값을 의미한다.
  map.merge(key, 1, (count, incr) -> count + incr);

  // 위의 로직을 메서드 참조를 사용해 간단하게 나타낼 수 있다.
  map.merge(key, 1, Integer::sum);
  ```

  > 📌 **참고**
  >
  > - 람다의 파라미터 리스트와 똑같은 구조의 파라미터를 받는 메서드라면 메서드 참조로 변경할 수 있다.
  >
  > - 즉 파라미터가 여러 개여도 메서드 참조를 쓸 수 있다.

<br>

### 람다식과 메서드 참조

- 람다식은 일회성이기 때문에 메서드를 작성하여 기능을 나타내는 이름을 지어줄 수도 있고 문서화를 할 수도 있다.

- 단, 무조건 메서드 참조가 좋은 것은 아니며 경우에 따라 더욱 간결하고 명확하게 나타낼 수 있는 방법을 사용하면 된다.

<br>

### 메서드 참조의 유형 5가지

- 정적 메서드를 참조하는 유형

  - 정적 메서드를 참조한다.

    ```java
    // 예시
    Integer::parseInt

    // 같은 기능을 수행하는 람다식
    str -> Integer.parseInt(str)
    ```

- 한정적 (인스턴스) 메서드 참조하는 유형

  - 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 동일하다.

    ```java
    // 예시
    Instant.now()::isAfter

    // 같은 기능을 수행하는 람다식
    // 파라미터로 받는 t가 참조되는 메서드인 then.isAfter의 파라미터로 들어간다.
    Instant then = Instant.now();
    t -> then.isAfter(t)
    ```

- 비한정적 (인스턴스) 메서드를 참조하는 유형

  - 함수 객체를 적용하는 시점에 수신 객체를 알려준다.

  - 람다식의 파라미터 리스트의 첫번 째에 수신 객체 전달용 매개변수가 선언된다.

  - 그 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 선언된다.

    ```java
    // 예시
    String::toLowerCase

    // 같은 기능을 수행하는 람다식
    // str을 받아서 str로 작업한다.
    str -> str.toLowerCase();
    ```

- 클래스 생성자를 참조하는 유형

  - 클래스를 생성할 때 기본 생성자를 사용하는 경우 사용한다.

    ```java
    // 예시
    TreeMap<K, V>::new

    // 같은 기능을 수행하는 람다식
    () -> new TreeMap<K, V>()
    ```

- 배열 생성자를 참조하는 유형

  - 배열을 생성할 때 사용한다.

    ```java
    // 예시
    int[]::new

    // 같은 기능을 수행하는 람다식
    len -> new int[len]
    ```

    <br>

### 메서드 참조 참고하기

- 메서드 참조 또한 람다와 같이 만능은 아니다. 메서드 참조가 더 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않으면 람다식을 사용하자.
