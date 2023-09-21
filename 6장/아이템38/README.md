# 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## 열거 타입은 상속이 불가능하다.

- 열거 타입은 묵시적으로 final 처리가 되므로 상속할 수 없다.

- 즉, 열거 타입의 값을 그대로 가져온 다음, 값을 더 추가하여 다른 목적으로 쓸 수 없다는 뜻이다.

- 원래 열거 타입은 상속을 하지 않는 것을 고려하여 설계하였으므로 실수로 이렇게 설계한 것은 아니지만, 확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있다.

<br>

## 연산 코드를 열거 타입을 활용하여 확장하자.

- 연산 코드는 기계가 수행하는 연산을 뜻한다. (+, - 등등..)

- 열거 타입 자체는 확장 (상속)을 할 수 없지만 열거 타입은 임의의 인터페이스를 구현할 수 있으므로 이 사실을 이용한다.

```java
// 인터페이스 선언
public interface Operation {
    double apply(double x, double y);
}

// 열거 타입의 각 상수는 Operation 인터페이스를 구현해야 한다.
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

- 앞의 BasicOperation를 상속할 수는 없지만, Operation 인터페이스를 활용하여 또 다른 열거형 타입으로 확장할 수 있다.

- 예를 들어 앞의 연산 타입을 확장해 지수 연산 (EXP)과 나머지 연산 (REMAINDER)을 추가해보자

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

<br>

## 테스트 해보기

### Enum이면서 Operation 타입인 파라미터만 받기

```java
public static void main(String[] args) {
    double x = Double.parseDouble("11");
    double y = Double.parseDouble("10");
    test(BasicOperation.class, x, y);
}

// 제네릭을 사용하여 enum 타입이면서 Operation 인터페이스 타입만 파라미터로 받는다.
private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

<br>

### Operation의 collection 타입인 파라미터만 받기

```java
public static void main(String[] args) {
    double x = Double.parseDouble("11");
    double y = Double.parseDouble("10");
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

// 꼭 enum이 아니더라도 Operation 인터페이스를 구현한 구현체의 Collection을 파라미터로 받는다.
private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

<br>

## 인터페이스를 사용한 enum 타입의 한계

- 열거 타입끼리 구현을 상속할 수 없다.

- 인터페이스를 구현하는 열거 타입끼리 공유하는 기능이 많으면 그 부분은 중복이 된다.

  - 중복이 발생하면 중복되는 로직을 처리하는 별도의 도우미 클래스나 `정적` 도우미 메서드로 코드 중복을 없앨 수도 있다.

- java 라이브러리 중에서도 java.nio.file.LinkOption 열거 타입은 CopyOption과 OpenOption 인터페이스를 구현한 라이브러리 중 1개이다.

> **📌 원준 의견**<br>
>
> - 인터페이스를 사용한 열거형 타입의 확장을 사용할 때 `기능을 확장하는 것이 아닌 다형성을 사용해서 여러 타입을 받을 수 있다는 것` 에 초점을 맞추어 설계하는 것이 좋아보인다.
> - 왜냐하면 열거형 타입끼리 상속이 불가능하므로 특정 1개의 열거형 타입의 기능을 확장하는 것이 불가능하기 때문이다.
