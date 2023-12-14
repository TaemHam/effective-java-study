# 아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

- 대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다.

- ordinal 클래스는 열거 타입의 상수가 몇번 째 상수인지 반환하는 메서드이다.

```java
public enum Ensemble {
    // ordinal 메서드를 사용하여 각 상수가 몇번 째로 선언되었는지 확인할 수 있다.
    SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}

public static void main(String[] args) {
    Arrays.asList(Ensemble.values())
        .forEach(elem -> System.out.println(elem + " >> " + elem.numberOfMusicians()));
}

/*  출력값은 아래와 같다.
    SOLO >> 1
    DUET >> 2
    TRIO >> 3
    QUARTET >> 4
    QUINTET >> 5
    SEXTET >> 6
    SEPTET >> 7
    OCTET >> 8
    NONET >> 9
    DECTET >> 10
*/
```

## ordinal을 사용하면 안되는 이유

- ordinal은 절대적인 값이 아니어서 상수의 선언 순서를 변경하면 각 상수에서 ordinal 메서드를 호출했을 때의 값도 바뀐다.

- 그리고 값을 중간에 건너뛸 수도 없다.

  - 예를 들어 각 상수가 1번, 2번, 4번, 5번을 가져야 할 때 ordinal 메서드를 사용할 수 없다.

## ordinal을 사용하지 말고 인스턴스 필드를 사용하자.

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4),
    QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8),
    DOUBLE_QUARTET(8), NONET(9),  DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int numberOfMusicians) {
        this.numberOfMusicians = numberOfMusicians;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```

- 위와 같이 각 상수에 번호를 부여함으로써 특정 상수가 삭제되거나 추가되어도 번호 값은 변하지 않는다.

- ordinal 메서드는 EnumSet이나 EnumMap과 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었으므로 사용하지 말자.
