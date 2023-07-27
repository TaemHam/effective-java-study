# 아이템 11. equals를 재정의하려거든 hashcode도 재정의하라

### equals를 재정의한 클래스 모두에서 hashcode도 재정의해야 한다.
- 그렇지 않으면 hashcode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킨다.
- 다음은 Object 명세에서 발췌한 규약이다.

    > 📌 **hashcode 규약**
    >
    > - equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
    >
    > - equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
    >
    > - equals(Object)가 두 객체를 다르다고 판단 했더라도 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블의 성능이 좋아진다.

### 좋은 해시코드를 작성하는 방법
1. int 변수 result를 선언한 후 c로 초기화한다.
 
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    - 기본 타입 필드라면 Type.hashCode(f)를 수행한다.
  
        ```java
        Integer.hashCode(20);

        Boolean.hashCode(true);

        Byte.hashCode(Byte.MAX_VALUE);

        Arrays.hashCode(new String[] {"val1", "val2"});

        ```

    - 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 이 필드의 표준형을 만들어 그 표준형의 hashCode를 사용한다.
      - 필드의 값이 null이면 0을 사용한다.
    
    - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음 3번 방법을 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashcode를 사용한다.
  
3. result = 31 * result + c;
4. result를 반환한다.

    > 📌 **hashcode 만들기 간단 정리!**
    >
    > - 해시코드를 만들 때 사용하는 데이터는 논리적으로 핵심 필드의 값을 사용하여 계산한다.
    >
    > - 핵심 필드가 아닌 파생 필드는 해시코드 계산에서 제외한다.
    > - equals 비교에 사용되지 않은 필드는 "반드시" 제외한다.
    >   - 그렇지 않으면 hashCode의 규약 중 2번을 어기는 것이다.

    <br>

    [왜 31을 사용할까?](https://velog.io/@indongcha/hashCode%EC%99%80-31)

    <br>

### 해시코드의 생성 비용이 바싸다면 캐싱을 하여 사용할 수 있다.

- 아래는 hashCode 메소드를 호출할 때마다 재계산 하는 것이 아닌 캐싱해서 사용하는 예제이다.

    ```java
    private int hashCode;

    @Override
    public int hashCode() {
        int result = hashcode;

        if(result == 0) {
            // 해시 코드 계산..
        }

        return result;
    }
    
    ```

    > 📌 **hashcode 생성 시 주의점**
    >
    > - hashCode 생성 비용이 비싸다고 해서 핵심 필드를 빼고 계산하면 안된다.
    >
    > - hashCode를 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
    >   - 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

    <br>

    > 📌 **Lombok의 @EqualsAndHashCode를 사용할 때 주의점**
    >
    > - hashCode는 논리적으로 핵심 필드로만 계산하는 것이 권장된다.
    >   - 예를 들어 각 entity의 id 값이 있다.
    >
    > - 그러나 Lombok의 @EqualsAndHashCode에서는 클래스 안에 선언된 모든 필드를 바탕으로 hashCode를 계산한다. 
    >
    > - 이 경우 2가지 단점을 생각할 수 있다.
    >   - 파생 필드까지 포함하여 hashCode를 계산하므로 비효율 적이다.
    >   - 논리적으로 동일한 객체로 판단하는 기준에 파생 필드가 포함되지 않는 경우, 논리적으로 같은 객체임에도 hashCode가 다르게 계산될 수 있다.


<br>

### 그렇다면 HashMap은 hashcode를 어떻게 활용하는 것일까?
- HashMap에서는 내부적으로 구현된 해시함수와 key의 hashCode 메소드를 가지고 key의 해시값을 계산한다.
  
- 모든 값에 대해 해시값을 다르게 부여하는 것은 불가능하므로 HashMap에서는 다음과 같은 방법을 사용한다.
  
  - 분리 연결법 & 보조 해시함수를 사용한다.
  - 분리 연결법에서는 해시 버킷이라는 개념을 사용하고 보조 해시함수로는 HashMap 클래스 안에 구현되어있다.


    [자바 7에서 HashMap은 어떻게 동작하는가?](https://d2.naver.com/helloworld/831311)

    [자바 17에서 HashMap은 어떻게 동작하는가?](https://howtodoinjava.com/java/collections/hashmap/how-hashmap-works-in-java/)

    [자바 17에서 HashMap의 내부구조](https://howtodoinjava.com/java/collections/hashmap/how-hashmap-works-in-java/)

    ```java
    public static void main(String[] args) {
		// key1과 key2는 해시코드가 2112로 동일하다.
		// 그런데 HashMap에서는 어떻게 같은 해시코드를 가지는
		// 2개의 값을 각기 다른 키로 구분하는 것일까?
		String key1 = "Aa";
		String key2 = "BB";

        // hashcode = 2112
		System.out.println("key1.hashCode() = " + key1.hashCode()); 

        // hashcode = 2112
		System.out.println("key2.hashCode() = " + key2.hashCode());

		HashMap<String, String> map = new HashMap<>();

		map.put(key1, "val1");
		map.put(key2, "val2");

		System.out.println(map.size()); // size = 2
	}
    ```