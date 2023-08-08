# 아이템 14. comaparable을 구현할지 고려하라.

### Comparable 인터페이스는 객체 간 비교하는 방법을 제공한다.
- Comparable 인터페이스의 compareTo 메소드는 단순 동치성 비교에 더해 순서까지 비교할 수 있다.

  ```java
  // 앞의 a와 뒤의 a가 같니? (동치성 비교)
  // 같다 -> 0 반환
  System.out.println("a".compareTo("a")); // 0 반환

  // 앞의 a와 뒤의 b가 같니?
  // 다르다 -> 그럼 앞의 a가 뒤의 b보다 순서가 빠르니? (순서 비교)
  // 느리다 -> -1
  System.out.println("a".compareTo("b")); // -1 반환

  // 앞의 a와 뒤의 b가 같니?
  // 다르다 -> 그럼 앞의 b가 뒤의 a보다 순서가 빠르니? (순서 비교)
  // 빠르다 -> 1 반환
  System.out.println("b".compareTo("a")); // 1 반환
  ```

  > 📌 **Comparable의 compareTo를 구현한다면 리턴값은 맘대로 줄 수 있다.**
  > - 반드시 0, -1, 1 중 하나를 반환해야 하는 것은 아니다.

  > - `동치성, 순서가 빠름, 순서가 느림` 의 3가지 상태만 나타낼 수 있는 구분값이면 된다.
  
  > - 그러나 Comparable을 구현하는 일반 규약이 있으니 되도록이면 규약을 따르도록 하자.


<br>

### 자바의 대부분의 클래스와 열거타입은 이미 Comparable을 구현해 놓았다.

- Arrays.sort() 를 사용해 정렬 시 compareTo 메소드가 사용된다.

- Arrays.sort()를 사용하여 배열의 일부만 정렬할 수 있다.

- Comparator나 Collections 인터페이스를 사용하여 클래스에서 Comparable을 구현하지 않고도 비교할 수 있다.
  
  
  ```java
  int[] arr = {1, 2, 3, 4, 5};

  // int 배열 정순 정렬
  Arrays.sort(arr);

  // int 배열 역순 정렬 (boxing이 되어야 API로 처리 가능)
  Collections.reverse(Arrays.asList(arr));
  ```
  ```java
  int[] arr = {1, 2, 3, 4, 5};

  // int 배열 정순 정렬
  Arrays.sort(arr);

  // int 배열 역순 정렬 (boxing이 되어야 API로 처리 가능)
  Collections.reverse(Arrays.asList(arr));

  // 우선순위 큐로 최대 힙 구현
  PriorityQueue<Integer> queue = new PriorityQueue<>(Comparator.reverseOrder());
  ```

<br>

### Comparable의 일반 규약

> 📌 **규약을 읽기 전 참고사항**
  > - `이 객체`란 compareTo를 호출하는 객체이며, `주어진 객체`란 compareTo에 파라미터로써 전달되는 객체를 의미한다.
  >   - `[이 객체]`.compareTo(`[주어진 객체]`)
  > - sgn (표현식) 표기는 수학에서 말하는 부호를 뜻하며 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.

- 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
  - `순서를 바꾸어 비교해도 같은 결과가 나와야 한다.`
 
- Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0) 이면 x.compare(z) > 0 이다.

- Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0 이면 sign(x.compareTo(z)) == sign(y.compareTo(z)) 입니다.
  - `즉 x == y라면, x.compareTo(anyObject)의 결과와 y.compareTo(anyObject)의 결과가 같아야 한다는 뜻이다.`

- 이번 권고는 필수는 아니지만 꼭 지키는 것이 좋다. (x.compareTo(y) == 0) == (x.equals(y))여야 한다.
  - x, y 객체가 있을 때 compareTo로 같다고 판단한다면 equals로도 같다고 판단할 수 있어야 한다는 뜻이다.
  
  - 만약 이 규약을 지키지 않으면 클래스에 이 규약을 지키지 않았다고 아래 문구처럼 써주면 될 것이다.
  
  - `주의: 이 클래스의 순서 (compareTo로 비교한 결과)는 equals 메서드의 결과와 일관되지 않다.`
  
  - 이 규약을 지키지 않은 경우 프로그램은 동작하나, 이미 정의된 Set, Map, Collection에 정의되어있는 compareTo나 equals를 사용하는 메소드에서 오류가 생길 수 있다. 
  
    - 예를 들어 HashMap이 있는데 A라는 메소드에서 compareTo로 2개의 객체를 같다고 판단했다고 하자. 그런데 B라는 메소드에서는 equals를 활용하여 다른 객체라고 판단할 경우 클라이언트가 원하는 결과가 나오지 않을 수 있다.
  
    - 실제로 BigDecimal 클래스는 compareTo와 equals가 일관되지 않다.

<br>

- 위의 규약은 equals의 규약과 매우 비슷하다.
- 상속을 받은 부모, 자식 클래스 관계 시 euqals 규약을 지키기 어려웠던 것처럼 compareTo도 똑같다.

  [toString의 규약 확인하기](https://github.com/TaemHam/effective-java-study/tree/main/3%EC%9E%A5/%EC%95%84%EC%9D%B4%ED%85%9C10)

<br>

### Comparable 작성 요령

- 대부분 equals의 작성요령과 비슷하다.

- Comparable은 제네릭 인터페이스이므로 컴파일 시 파라미터 타입이 정해진다.

- 그래서 파라미터 타입을 확인할 필요가 없는 점이 equals와 다르다.

<br>

#### 작성 요령 1. Comparable를 이용하여 순서를 비교한다.
- comapareTo는 객체의 동치성이 아닌 `순서`를 비교하는 것이기 때문에 객체의 참조 필드를 비교하려면 참조 필드의 내부도 compareTo로 계속 확인해야 한다. (compareTo 메서드를 사용해서 재귀적으로 비교한다는 것이다.)

- Comparable를 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 (역순, 내가 원하는 정렬 순서 등) Comparator 인터페이스를 활용하여 새로 구현하거나, 자바가 제공하는 API를 쓰자.

  ``` java
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
  }
  ```
- 박싱 필드, String에도 compareTo가 있으므로 활용하자.
  ``` java
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
  }
  ```
  
#### 자바 7 이후부터 박싱된 기본 타입에는 compareTo 메소드가 있으므로 그것을 활용하자.
- 박싱된 기본타입의 compare 메서드 예제
  
  ``` java
  Integer.compare(1, 2);

  Float.compare(1, 2);

  Double.compare(1, 2);

  Boolean.compare(true, false);
  ```

#### 클래스에 핵심 필드가 여러 개라면 그 중에서도 가장 핵심 필드부터 비교해나가자.
- equals와 거의 비슷한 구조이다.
  
  ``` java
  public class PhoneNumber implements Comparable<PhoneNumber> {
      @Override
      public int compareTo(PhoneNumber phoneNumber) {
          int result = Short.compare(areaCode, phoneNumber.areaCode); // 가장 중요한 필드
          if (result == 0) {
              result = Short.compare(prefix, phoneNumber.prefix); // 두 번째로 중요한 필드
              if (result == 0) {
                  result = Short.compare(lineNum, phoneNumber.lineNum); // 세 번째로 중요한 필드
              }
          }
          return result;
      }
  }
  ```

#### 자바 8부터 메서드 체이닝 방식으로 Comparator를 생성할 수 있게 되었다.
- 약간의 성능 저하가 있긴 하지만 코드가 깔끔해진다는 장점이 있다.
  ``` java
  private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
            .thenComparingInt(pn -> pn.prefix)
            .thenComparingInt(pn -> pn.lineNum);

  public int compareTo(PhoneNumber pn) {
      return COMPARATOR.compare(this, pn);
  }
  ```

  > 📌 **메서드 체이닝으로 Comparator를 사용할 때 성능저하에 관하여**
  > - 위의 예제처럼 총 3개의 비교 메서드가 체이닝 되어있다고 하자.
  > - 이러한 경우 첫번 째 메소드 실행 후, 두번 째, 세번 째 메소드 순서로 실행된다. 중간에 원하는 정렬이 만족되어도 마지막 연산까지 하므로 성능이 약간 저하되는 것 같다. (쇼트 서킷이 적용되지 않음)

  > - 그러나 구글링 시 성능 저하에 대한 자료가 거의 나오지 않는 것으로 보아 실제로는 미미한 성능 저하라고 생각되며 가독성이 높아진다는 점에서 메서드 체이닝으로 작성하는 것이 좋을 것 같다는 의견이다.

  <br>
  
  [Comparator 활용법 정리 블로그](https://veneas.tistory.com/entry/Java-%EC%9E%90%EB%B0%94-8-Comparator-API-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%A0%95%EB%A0%AC)