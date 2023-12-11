# 아이템 75. 예외의 상세 메시지에 실패 관련 정보를 담으라

## 스택 추적 (Stack trace)

예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 그 예외의 스택 추적 정보를 자동으로 출력한다. 

스택 추적은 예외가 발생했을 때 프로그램이 실행 중에 호출한 메소드의 리스트로, 일반적으로 예외 객체의 `toString()` 메서드를 호출해 얻는 문자열이고, 형식은 다음과 같다.

<details>
<summary>`예외의 클래스 이름 + 상세 메시지`</summary>

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
</details>
<br>


스택 추적은 실패 원인을 분석하는 데 있어 유일한 정보이므로, 예외의 `toString()` 메서드에 최대한 많은 정보를 담아 반환하는 것이 중요하다.

### 규칙

다음은 스택 추적을 정보에 어떤 정보를 담아야 하는지에 관해 참고할만한 규칙들이다.

1. **실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.** 

만약 IndexOutOfBoundsException 의 상세메시지는 범위의 최솟값, 최댓값, 그 범위를 벗어났다는 인덱스의 값을 담아야 한다. 해당 에러가 발생할 원인들은 매우 다양하므로, 해당 정보를 보면 무엇을 고쳐야 할지를 분석하는데 큰 도움이 된다.

2. **관련 데이터를 모두 담아야 하지만 장황할 필요는 없다.**

문제를 분석하는 사람은 스택 추적뿐만 아니라 관련 문서와, 소스코드를 함께 살펴보기 떄문에 여기서 얻을 수 있는 정보를 길게 늘어놔봐야 군더더기가 될 뿐이다.

3. **예외의 상세 메시지와 최종 사용자에게 보여줄 오류 메시지를 혼동해서는 안된다.**

최종 사용자에게는 친절한 안내 메시지를, 예외 메시지는 가독성보다는 담긴 내용을 중요시 해서 보여주여야 한다.

4. **실패를 적절히 포착하려면 필요한 정보를 예외 생성자에서 모두 받아서 상세 메시지까지 미리 생성해놓는 방법도 좋다.**

예를 들어, 현재의 `IndexOutOfBoundException`은 이미 생성된 문자열 String이나, 호출할 때 사용한 인덱스 int 를 받는 생성자가 존재한다.

```Java
public class IndexOutOfBoundsException extends RuntimeException {
    private static final long serialVersionUID = 234122996006267687L;
    public IndexOutOfBoundsException() {
        super();
    }
    public IndexOutOfBoundsException(String s) {
        super(s);
    }
    public IndexOutOfBoundsException(int index) {
        super("Index out of range: " + index);
    }
}
```

하지만 추가적으로 호출할 수 있는 최소값과 최대값 정보를 포함시킬 수 있었어도 좋았을 것이다.

```JAVA
    /**
      * IndexOutOfBoundsException을 생성한다
      * 
      * @param lowerBound 인덱스의 최솟값
      * @param upperBound 인덱스의 최댓값 + 1
      * @param index 인덱스의 실젯값
      */
    public IndexOutOfBoundsException(int lowerBound, int upperBound, int index){
        // 실패를 포착하는 상세 메시지를 생성한다.
        super(String.format("최솟값: %d , 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
        
        // 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다.
        this.lowerBound = lowerBound;
        this.upperBound = upperBound;
        this.index = index;
    }
```

5. **예외는 실패와 관련된 정보를 얻을 수 있는 접근자 메서드를 적절히 제공하는 것이 좋다.**

포착한 실패 정보는 예외 상황을 복구하는데 유용할 수 있으므로, 접근자 메서드는 `uncheckedException` 보다는 `checkedException` 에서 더 빛을 발한다. 

하지만 'toString이 반환 값에 포함된 정보를 얻어올 수 있는 API 를 제공하자' 는 일반원칙을 따른다는 관점에서 비검사 예외라도 상세 정보를 알려주는 접근자 메서드를 제공하는 것이 좋다.

