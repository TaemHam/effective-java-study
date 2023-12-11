# 아이템 73. 추상화 수준에 맞는 예외를 던지라

메서드가 저수준 예외, 즉 구체화 단계의 예외를 처리하지 않고 바깥으로 던져버리면<br>
내부 구현 방식을 드러내어 윗 레벨 API 를 오염시킨다.<br>
다음 릴리스에서 구현 방식을 바꾸면 다른 예외가 튀어나와, 프로그램을 깨지게 할 수 있는 것.

이를 해결하기 위해선, **상위 계층에서 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다.**

이를 **예외 번역(Exception translation)** 이라고 한다.

다음은 `AbstractSequentialList`에서 수행하는 예외 번역의 예시이다.
```Java
public E get(int index){
	ListIterator<E> i = listIterator(index);
    try{
    	return i.next();
    }catch(NoSuchElementException e){
    	throw new IndexOutOfBoundsException("인덱스 : " + index);
    }
}
```

예외를 번역할 때, **저수준의 예외를 고수준의 예외로 실어보낼 수도 있다**.

이를 **예외 연쇄(Exception chaining)** 라고 한다.

예외 연쇄 방식을 사용하면 예외 로그가 인과가 순서대로 쌓여 추적이 쉽다는 장점이 있다.

다음은 [Baeldung 블로그](https://www.baeldung.com/java-chained-exceptions)에서 가져온, 예외 연쇄 방식을 사용해 예외를 발생시키는 코드이다.

```Java
public class MainClass {
    public void main(String[] args) throws Exception {
        getLeave();
    }

    public getLeave() throws NoLeaveGrantedException {
        try {
            howIsTeamLead();
        } catch (TeamLeadUpsetException e) {
             throw new NoLeaveGrantedException("Leave not sanctioned.", e);
        }
    }

    public void howIsTeamLead() throws TeamLeadUpsetException {
        throw new TeamLeadUpsetException("Team lead Upset.");
    }
}
```

위를 실행 시키면 다음과 같은 로그가 나온다.

```
Exception in thread "main" com.baeldung.chainedexception.exceptions
  .NoLeaveGrantedException: Leave not sanctioned. 
    at com.baeldung.chainedexception.exceptions.MainClass
      .getLeave(MainClass.java:36) 
    at com.baeldung.chainedexception.exceptions.MainClass
      .main(MainClass.java:29) 
Caused by: com.baeldung.chainedexception.exceptions
  .TeamLeadUpsetException: Team lead Upset.
    at com.baeldung.chainedexception.exceptions.MainClass
  .howIsTeamLead(MainClass.java:44) 
    at com.baeldung.chainedexception.exceptions.MainClass
  .getLeave(MainClass.java:34) 
    ... 1 more
```

<details>
<summary>만약 예외 연쇄 방식을 사용하지 않는다면...</summary>

```Java
public class MainClass {

    public void main(String[] args) throws Exception {
        getLeave();
    }

    void getLeave() throws NoLeaveGrantedException {
        try {
            howIsTeamLead();
        } catch (TeamLeadUpsetException e) {
            e.printStackTrace();
            throw new NoLeaveGrantedException("Leave not sanctioned.");
        }
    }

    void howIsTeamLead() throws TeamLeadUpsetException {
        throw new TeamLeadUpsetException("Team Lead Upset");
    }
}
```

```
com.baeldung.chainedexception.exceptions.TeamLeadUpsetException: 
  Team lead Upset
    at com.baeldung.chainedexception.exceptions.MainClass
      .howIsTeamLead(MainClass.java:46)
    at com.baeldung.chainedexception.exceptions.MainClass
      .getLeave(MainClass.java:34)
    at com.baeldung.chainedexception.exceptions.MainClass
      .main(MainClass.java:29)
Exception in thread "main" com.baeldung.chainedexception.exceptions.
  NoLeaveGrantedException: Leave not sanctioned.
    at com.baeldung.chainedexception.exceptions.MainClass
      .getLeave(MainClass.java:37)
    at com.baeldung.chainedexception.exceptions.MainClass
      .main(MainClass.java:29)
```
</details>
<br>


예외 연쇄용 생성자는 다음과 같이 `super()`를 이용해 상위 생성자를 호출해 만들면 된다.

```Java
class NoLeaveGrantedException extends Exception {

    public NoLeaveGrantedException(String message, Throwable cause) {
        super(message, cause);
    }

    public NoLeaveGrantedException(String message) {
        super(message);
    }
}

class TeamLeadUpsetException extends Exception {
    // Both Constructors
}
```


**무턱대고 예외를 전파하는 것보다 예외 번역이 우수한 방법이지만, 그렇다고 남용하면 곤란하다.** 가능하다면 저수준 메서드가 반드시 성공하도록 하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다.