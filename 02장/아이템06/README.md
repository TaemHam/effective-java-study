# 아이템 06. 불필요한 객체 생성을 피하라

### 똑같은 기능의 객체를 매번 생성하기 보다는 객체 하나를 재사용하는 편이 나을 때가 많다.
- String 을 선언할 때는 리터럴 문자를 넣어주자.

    ```java
    String str = new String("myString"); // 사용하지 말아야 한다!!

    String str = "myString"; // 이 방법을 사용하자.
    ```
    > 📌 **왜 위의 첫 번째 방법을 사용하지 말야아 하나?**
    >
    > - 자바에서 new String()을 사용하면 매번 String 인스턴스를 새로 만들기 때문이다.

<br>

- 생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용하자.

    ```java
    Boolean bool = Boolean.valueOf("true");
    ```
    > 📌 **Boolean.valueOf는 왜 불변 객체를 생성한다는 걸까?**
    >
    > - "true".equalsIgnoreCase(s)로 작성되어있는데, "true" 문자열이 불변이기 때문이다.

<br>

- 생성 비용이 아주 비싼 객체도 여러 번 생성하지 않고 캐싱하여 재사용하길 권장한다.

    ```java
    public class RomanNumerals {
        private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})");

        static boolean isRomanNumeral(String s) {
            return ROMAN.matcher(s).matches();
        }
    }
    ```
    > 📌 **위의 코드가 동작하는 방식은?**
    >
    > - RomanNumerals 클래스가 로딩될 때 static 변수인 ROMAN에 값을 할당 후, isRomanNumeral 메소드가 호출될 때마다 ROMAN 변수를 새로 생성하지 않고 가져다가 사용하는 것이다.



<br>

- 객체가 불변이라면 재사용해도 안전하다. 그러나 직관에 반대되는 상황도 있다.
    - 어댑터는 실제 작업은 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 하는 객체이다.

    - 어댑터는 여러 개일 필요가 없으므로 어댑터를 호출할 때 매번 같은 어댑터 객체를 사용하는 것이 이득이다.
    ```java
	public static void main(String[] args) {
		Map map = new HashMap();

		map.put("key1", "value1");
		map.put("key2", "value2");

		Set set1 = map.keySet();
		Set set2 = map.keySet();

		System.out.println(set1 == set2);
	}
    ```
    [어댑터 패턴에 대한 설명](https://johngrib.github.io/wiki/pattern/adapter/)

<br>

- 불필요한 객체를 만들어내는 또 다른 예로 오토박싱이 있다.
    - 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만 완전히 없애주는 것은 아니다.
    - 그러므로 박싱된 기본타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

    - 아래 코드는 매우 비효율적이므로 박싱된 기본타입보다 기본 타입을 사용하는 것이 훨씬 빠르다.
    ```java
    private static long sum() {
		Long sum = 0L;
		for (int i = 0; i <= Integer.MAX_VALUE; i++) {
			sum += i;
		}
		return sum;
	}
    ```
    > 📌 **JPA 엔티티의 컬럼 필드는 무조건 Wrapper Class로 선언해야 할까?**
    >
    > - 반드시 그럴 필요는 없다!
    > - 다만 null 값으로 엔티티 간 연관관계를 나타낼 수 있는 부분이 필요하다면 Wrapper Class를 써야 한다.
    > - 또는 null로 엔티티의 상태를 나타낼 수 있다면 Wrapper Class를 써야 한다.

[엔티티 필드에 반드시 Wrapper Class를 사용해야 할까?](https://velog.io/@d-h-k/JPA-Entity-Class-%EC%97%90%EC%84%9C-Primitive-Type-%EC%9D%84-%EC%8D%A8%EC%95%BC%ED%95%A0%EA%B9%8C-Wrapper-Class-%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC%ED%95%A0%EA%B9%8C)

[엔티티의 PK에 반드시 Wrapper Class를 사용해야 할까?](https://stackoverflow.com/questions/9146967/should-i-use-primitives-or-wrappers-in-jpa2-0)

<br>

- 그렇다고 객체 생성은 비싸니까 무조건 객체 풀을 만들어야 하는 것은 아니다!
<br>
<hr>

## 추가 학습
### String Constant Pool

- String Constant Pool은 리터럴 문자를 저장하는 영역이다. 메모리의 어느 지점에 저장되는 지는 JVM 벤더의 구현체마다 다를 수 있다.

    ```java
    String str1 = "a";
    String str2 = "a";

    System.out.println(str1 == str2); // true를 반환한다.
    ```

[String Constant Pool 알아보기](https://stackoverflow.com/questions/4918399/where-does-javas-string-constant-pool-live-the-heap-or-the-stack)

<br>

### 정규식

- 정규식이란 프로그래밍에서 문자열을 다룰 때, 문자열의 일정한 패턴을 표현하는 일종의 형식 언어를 말한다.
- 그러나 정규식은 "백트래킹" 방식을 사용하므로 성능이 조금 느린 부분이 있다.

[정규식을 웹에서 사용하기](https://regexr.com/)

<br>

### Pattern 클래스

- Pattern 클래스는 정규 표현식이 컴파일된 클래스. 정규 표현식에 대상 문자열을 검증하거나, 활용하기 위해 사용되는 클래스이다. 

<br>

### 스프링에서 제공하는 패턴 매칭 util 클래스

- PatternMatchUtils
    - PatternMatchUtils는 패턴과 패턴에 매칭하는지 판단할 문자열로 패턴 매치 여부를 체크한다.


    ```java
    String[] patterns = {"/manager/**", "/admin/*"};

    boolean isMatched1 = PatternMatchUtils.simpleMatch(patterns, "/manager/myPage");
    System.out.println(isMatched1); // true 반환

    boolean isMatched2 = PatternMatchUtils.simpleMatch(patterns, "items");
    System.out.println(isMatched2); // false 반환


- AntPathMatcher
    - PatternMatchUtils와 다르게 AntPathMatcher "경로 문자열"에 대한 매칭여부를 판단한다.
    - AntPathMatcher는 경로에 와일드 카드를 주고, 패턴에 매칭하는지 판단할 문자열로 패턴 매치 여부를 체크한다.

    - PatternMatchUtils와 다르게 패턴을 배열이 아닌 1개만 전달하도록 되어있다.

    ```java
    AntPathMatcher matcher = new AntPathMatcher();

    boolean isMatched = matcher.match("/manager/*", "/manager/myPage");

    System.out.println(isMatched); // true 반환
    ```
    > 📌 **PatternMatchUtils는 정적 메소드를 사용하는데 AntPathMatcher는 인스턴스를 생성하는 이유?**
    >
    > - PatternMatchUtils는 클래스에 상태값을 가지지 않는다.
    >   - 그러므로 simpleMatch 정적 메소드를 선언하여 메모리를 절약할 수 있다.
    >   - AntPathMatcher와 다르게 pathSeparator를 가지지 않으므로 경로 문자열 이외의 경우에도 사용 가능하다.

    > - 반면 AntPathMatcher는 클래스는 경로 문자열에 대한 match 여부를 확인한다.
    >   - 그러므로 반드시 pathSeparator를 가져야 하고, AntPathMatcher의 인스턴스마다 pathSeparator는 각기 다른 값을 가질 수 있다.
    >   - pathSeparator는 기본값을 가지거나 생성자 파라미터 or 수정자로 입력해 주어야 한다.
    


