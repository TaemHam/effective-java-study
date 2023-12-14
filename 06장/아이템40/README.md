# 아이템 40. @Override 애너테이션을 일관되게 사용하라

### @Override 애너테이션은 왜 사용할까?

상위 타입의 메서드를 재정의했음을 `컴파일러에게 알려` 사람이 내는 실수를 걸러내기 위함이다.

다음 클래스는 회원 정보 조회 결과를 담은 응답용 DTO이다. (책에 있는 예제보다, Spring 개발자에게 좀 더 와닿는 예제를 가지고 왔다.)

```JAVA
public class MemberDetailResponseDto {
    private String memberId;
    private String memberEmail;

    public MemberDetailResponseDto(String memberId, String memberEmail) {
        this.memberId = memberId;
        this.memberEmail = memberEmail;
    }

    public String getMemberId() {
        return this.memberId;
    }

    public String getMemberEmail() {
        return this.memberEmail;
    }

    public boolean equals(MemberDetailResponseDto m) {
        return this.memberId.equals(m.getMemberId()) && this.memberEmail.equals(m.memberEmail);
    }

    ...
}
```

그리고 이 DTO를 반환하는 서비스 로직의 테스트 코드를 짠다고 할 때, 실제 객체와 원하는 값을 가진 객체를 비교하는 `assertEquals()`를 사용해 다음과 같이 짤 수 있다.

```JAVA
...
// When
MemberDetailResponseDto actual = memberService.readProfile(requestDto);

// Then
MemberDetailResponseDto expected = new MemberDetailResponseDto(expectedId, expectedEmail);
assertEquals(actual, expected)
```

만약 서비스 로직이 잘 작동해 actual 과 expected에 담긴 값이 동일하다면, 이 테스트 코드는 성공할까?

아마 다음과 같은 오류를 내며 실패하게 될 것이다.

```
expected: <org.test.MemberDetailResponseDto@72af90e8> but was: <org.test.MemberDetailResponseDto@aa1bb14>
```

분명히 `equals()` 메서드를 재정의해 객체의 값을 비교하도록 만들었으나, 객체의 주솟값을 비교하고 있다.

이런 오류가 난 이유는 위에서 정의한 `equals()`가 매개변수로 `Object`를 받지 않고 `MemberDetailResponseDto`를 받아, `equals()`를 재정의하지 못했기 때문이다. 즉, Override가 아닌, Overload를 해버린 꼴이다.

이렇게 재정의할 때 사람이 내는 실수를 컴파일러가 잡아줄 수 있도록 하려면, @Override 애너테이션을 붙여주는 편이 좋다.

![IDE catches override error](https://github.com/TaemHam/effective-java-study/assets/95671168/fbb9cec1-f58b-444a-adac-18e3626e8ce5)

**따라서, 재정의한 모든 메서드에 `@Override` 애너테이션을 의식적으로 다는 습관을 가지도록 하자.**