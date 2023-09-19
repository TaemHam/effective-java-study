# 아이템 36. 비트 필드 대신 EnumSet을 사용하라

## 비트 필드

열거한 값들을 "여러 값 중 하나만 골라" 사용하는 게 아니라 "여러 값의 집합"으로 사용하고자 한다고 하자.

예전에는 각 값들을 서로 다른 2진수 비트 값을 주고, 해당 자리수의 비트가 0인지 1인지 판단하는 정수 열거 패턴을 사용했다.  

```JAVA
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) {
        // ...
    }
}
```

이렇게 처리하면 비트별 OR(`|`)연산을 통해 여러 상수의 집합을 표현할 수 있으며, 이렇게 만들어진 집합을 비트 필드(bit field)라 한다.

### 비트 필드의 장점

- 집합을 표현하는 데 int나 long 하나 만큼의 메모리 공간만 필요함
- 합집합, 교집합 등 집합 연산이 비트 연산으로 이루어져 매우 효율적임

### 비트 필드의 단점

- 정수 열거 패턴의 일종이므로, 정수 열거 패턴의 단점을 그대로 가지고 있음
    - 타입 안전을 보장할 수 없다.
    - 정수 열거 패턴을 위한 namespace를 제공하지 않기 때문에 변수명 충돌을 막을 수 없다.
    - 문자열 열거 패턴도 있는데 이 패턴은 정수 열거 패턴과 동일하게 클라이언트에서 어떤 문자열을 넣어도 컴파일 시 막아줄 수 없다.

- 어떤 상수가 집합에 포함되어있는지 한 눈에 보기 어렵고, 해독하는 과정이 필요함
- 집합에 포함된 원소를 순회하기도 까다로움
- 상수 종류가 몇 개인지에 따라 적절한 타입(32개 이하면 int, 아니면 long)

이렇게 많은 단점을 가지는 비트 필드의 대안은 무엇일까?

## EnumSet

java.util 패키지의 `EnumSet` 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

```JAVA
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 다형성을 위해 어떤 Set도 받을 수 있도록 설계했지만, EnumSet이 가장 좋다.
    public void applyStyle(Set<Style> styles) {
        // ...
    }
}
```

### EnumSet의 특징

- Set 인터페이스를 완벽히 구현
- 타입 안전
- 내부적으로 비트 벡터로 구현되어, 원소 종류가 64개 이하라면 전체 집합을 long 하나로 표현해 비트 필드에 비견되는 성능을 보장
- retainAll(교집합)이나 removeAll(차집합) 연산도 산술 연산을 사용해 비트 필드만큼 효율적이다
- EnumSet의 유일한 단점은 불변 클래스로 만들 수는 없다는 것이다.

이런 특징들을 고려할 때 대부분의 상황에서는 비트 필드보다 EnumSet을 사용하는 게 좋다. 