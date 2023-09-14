# 아이템 34. int 상수 대신 열거 타입을 사용하라

## 열거 타입을 잘 사용하는 법

### 상수마다 메서드를 구현하기

열거 타입으로 단순히 값을 저장하는 것 뿐만 아니라, 각 상수마다 특수한 기능도 더해주고 싶을 수 있다.

예시를 위해 연산 관련 상수를 다루는 Operation 열거 타입이 있다고 해보자.

```JAVA
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE
}
```

만약 두 수가 주어졌을 때 각 상수의 이름에 걸맞은 연산 값을 반환하는 기능을 추가하려면 어떻게 해야할까?

물론, switch 문으로 분기 처리하는 방법을 쓸 수도 있다.

```JAVA
public double apply(double x, double y) {
    switch(this) {
        case PLUS: return x + y;
        case MINUS: return x - y;
        case TIMES: return x * y;
        case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
}
```

구현은 간단하지만, 이 방법을 사용하면 OCP를 어기게 된다는 단점이 있다.<br>
예를 들어, 제곱 연산을 위해 POWER 라는 상수를 추가한다고 해보자.<br>
새로 추가된 연산을 적용하려면, apply 함수를 수정해야만 한다.

또, 실수로 수정하는 걸 잊어버리기라도 한다면, 런타임에 AssertionError를 내며 프로그램이 종료되는 상황이 생길 수 있다.

switch문보다 안전하게 구현하는 방법도 있는데, 바로 **추상 메서드를 선언하고 구현하는 방법**이다.

```JAVA
public enum Operation {
    PLUS {public double apply(double x, double y) {return x + y;}},
    MINUS {public double apply(double x, double y) {return x + y;}},
    TIMES {public double apply(double x, double y) {return x + y;}},
    DIVIDE {public double apply(double x, double y) {return x + y;}};
    
    public abstract double apply(double x, double y);
}
```

새로운 연산이 추가되어도 기존의 코드가 수정될 일이 없다.

`apply()`가 추상 메서드이므로, 잊어버리고 재정의하지 않았다면 컴파일 오류로 알려준다.<br>
사실 앞의 상수들에 메서드 구현이 붙어있으니, 잊어버리기도 쉽지 않다.

이렇게 메서드 구현부를 넣는 것도 좋지만, 추가적으로 데이터를 가지게 할 수도 있는데,  다음 예제 코드와 같이 특정 문자열 값으로 열거 타입 객체를 찾는 기능도 넣을 수 있다.

```JAVA
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {return x + y;}
    }
    MINUS("-") {
        public double apply(double x, double y) {return x - y;}
    }
    TIMES("*") {
        public double apply(double x, double y) {return x * y;}
    }
    DIVIDE("/") {
        public double apply(double x, double y) {return x / y;}
    };
    
    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public String getSymbol() {
        return symbol;
    }
    
    public abstract double apply(double x, double y);

    // symbol을 키로, 각 열거형을 밸류로 맵에 저장
    private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(toMap(Operation::getSymbol, e -> e));

    // Optional을 반환해 클라이언트에게 null 값이 나올수 있음을 알린다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
}
```

짚고 넘어가야 할 점이 있다면, `stringToEnum` 맵에 열거형 타입을 추가할 때, 생성자에서 쉽게 this 로 추가하지 않고 굳이 정적 필드 초기화하며 넣어주었다는 것이다.

구체적인 코드로 나타내보자면 다음과 같은 상황이다.

```JAVA
private static final Map<String, Operation> stringToEnum = new HashMap<>();

Operation(String symbol) {
    this.symbol = symbol;
    stringToEnum.put(symbol, this);
}
```

왜 이 코드처럼 간단하게 구현하지 않았을까?

이유는 열거 타입의 생성자 호출 시기때문이다.

열거 타입의 각 객체의 생성자는 정적 필드 초기화보다 더 일찍 호출된다.<br>
즉, Operation의 생성자에서 stringToEnum에 접근한다는 것은, HashMap이 만들어지기도 전에 HashMap에 접근하려고 하는 것과 같다.

때문에 위 코드를 실제로 컴파일 해보면 아래와 같은 오류 메시지가 나온다.

```
java: illegal reference to static field from initializer
```

### 전략 열거 타입 패턴 사용하기

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

예시로 급여명세서에서 쓸 요일을 표현하는 열거 타입을 생각해 보자.

```JAVA
enum PayrollDay {
    MONDAY, TUESDAY, WEDSDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}
```

월화수목금 -> 오버 타임만 잔업 수당 계산 / 토일 -> 모든 시간이 잔업 수당에 포함

위와 같은 기능을 추가하려 하는데, 위의 방식대로 추상 메서드를 선언하고 각각의 타입에 구현해 넣자니, 중복 코드가 너무 많아지는 상황이 발생했다.

중복 코드가 생기지 않게 하기 위해, switch 문으로 분기처리하도록 할 수도 있다.

```JAVA
enum PayrollDay {
    MONDAY, TUESDAY, WEDSDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        
        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY: // 주말
                overtimePay = basePay / 2;
                break;
            default: // 주중
                overtimePay = minutesWOrked <= MINS_PER_SHIFT ?
                0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
            
        return basePay + overtimePay;
    }
}
```

간결하긴 하지만, 역시 OCP를 위반한 코드이다.

휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case문을 잊지 말고 쌍으로 넣어줘야 한다. 만약 잊어버렸다면 평일에 근무하는 것처럼 계산되어 버린다.

이 역시 대체할 방법이 존재하는데, 바로 **전략 열거 타입 패턴**이다.

단어가 길어 어려워보이지만, 사실 간단하다.

다른 열거형 타입을 만들고, 분기처리할 코드( == 전략)마다 하나씩 타입을 선언하는 것이다. 

이를 위 열거타입에 적용하면 아래와 같은 코드가 된다.

```JAVA
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }

    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```

이제 휴가 같은 날이 추가 되어도, 단순히 PayType을 지정해 주기만 하면 되고, 만약 급여 산출 방식이 새로 생기거나 바뀌어도, PayType만 추가, 수정 해주면 되니, 유지 보수가 훨씬 편해졌다할 수 있다.

## 기타 열거 타입에 관한 팁

* 필요한 원소를 컴파일 타임에 알 수 있는 상수 집합이라면 열거 타입을 쓰자

* 상수 추가 때문에 바이너리 호환이 안될까봐 걱정하지 말자