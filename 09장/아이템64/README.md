# 아이템 64. 객체는 인터페이스를 사용해 참조하라

객체는 클래스가 아닌 인터페이스로 참조하라. 이 말은 무슨 뜻일까?

```Java
//좋은 예 
Set<String> set = new LinkedHashSet<>();

//나쁜 예
LinkedHashSet<String> set = new LinkedHashSet<>();
```

객체를 생성할 때 LinkedHashSet으로 객체를 만들지만 업캐스팅을 사용하고 있다. <br>
업캐스팅을 사용하게되면 상위 인터페이스에 있는 구현체를 사용할 수 있지만 LinkedHashSet만이 가진 메서드를 사용할 순 없다.

인터페이스를 사용하는 것이 좋은 이유는 바로 **유연성**에 있다.

예를 들어, 위의 `Set`의 구현체를 `HashSet`으로 교체한다고 해보자.

아래의 나쁜 예라면, 위 `LinkedHashSet`을 사용하는 곳 모두를 `HashSet`으로 변경해야 하겠지만, <br>
위의 좋은 예에서는, 선언하고 있는 곳의 구현체만 바꿔도 문제 없이 사용 가능하다.

```JAVA
Set<String> set = new HashSet<>();
```

### 인터페이스로 변경이 불가능 한 때

다음과 같은 적합한 인터페이스가 없는 경우 인터페이스로 변경이 불가할 수 있다.

* `String`, `BigInteger` 같은 값 클래스
* `OutputStream` 같은 클래스 기반으로 작성된 프레임워크가 제공하는 객체들

    ![OutputStream](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb54kFK%2FbtqN8dXJMUA%2FfYebStLGQERm1U0VxyrYW1%2Fimg.png)
    Closable이나 Flushable로 사용할 수는 없다.
    
* `PriorityQueue` 클래스의 `comparator` 메서드 같이 원래 클래스가 인터페이스에 정해진 기능 외의 특별한 기능을 제공하고, 다른 코드가 그 기능에 의존하는 경우
    
    클래스 타입을 직접 사용하는 경우는 이런 추가 메서드를 꼭 사용해야 하는 경우로 최소화해야 한다.