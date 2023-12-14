# 아이템 62. 다른 타입이 적절하다면 문자열 사용을 피하라

문자열은 범용성이 매우 뛰어나서 String 타입 외에도 다른 타입으로 사용하려는 경향이 있다.

이번 시간에는 문자열을 다른 타입으로 사용하지 말아야할 이유에 대해 다루려고 한다.

### 값 타입

파일, 네트워크, 키보드 입력 등 데이터를 받을 때 문자열로 받기 때문에, 문자열을 그대로 사용하기도 한다.

예를 들어, 숫자를 int, float, BigInteger 가 아닌 "1" 처럼 문자열로 받는다던가, 예/아니오 를 표현할 boolean을 "Y" 나 "N"으로 받는다던가 하는 경우가 있다.

받아서 사용하는 입장에서, 문자열을 처리하는 과정을 한 번 거치기 때문에 오류가 발생할 가능성이 생기고, <br>
또 문자열이 실제 타입보다 더 큰 메모리를 잡아먹는다는 단점도 있다.

### 열거 타입

문자열을 열거 타입으로 사용하지 않는 이유는 지난 [[아이템 34](https://github.com/TaemHam/effective-java-study/tree/main/6%EC%9E%A5/%EC%95%84%EC%9D%B4%ED%85%9C34)]에서도 다뤘듯, <br>
타입 안전을 보장할 수 없고, 변수명 충돌을 막을 수 없으며, 약타입이 된다는 문제점들이 있기 때문이다.

### 혼합 타입

여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것도 좋은 것은 아니다.

```JAVA
String compoundKey = className + "#" + i.next();
```

위의 코드처럼 두 요소를 "#"을 이용해 구분지어주는 방식도 단점이 많다.

1. 두 요소를 추출하기 위해 문자열을 파싱해야해 성능이 나빠진다.
2. "#"이 className 이나 i.next()에서 이미 사용중이라면, 파싱이 어려워진다.
3. 각각의 요소가 원래 가지고 있었을 타입에 대한 equals, toString, compareTo 메서드를 사용할 수 없고, String이 제공하는 기능에만 의존해야 한다.

이 경우는 차라리 전용 클래스를 새로 만들어 두 요소를 각각 저장하고, 필요하다면 toString() 메서드를 위 형식으로 만드는 방법을 사용하는 것이 좋다.

```JAVA
public class Compound {
    private CompoundKey compoundKey;

    private static class CompoundKey {
        private String className;
        private String delimiter;
        private int index;

        public CompoundKey(String className, String delimiter, int index) {
            this.className = className;
            this.delimiter = delimiter;
            this.index = index;
        }

        @Override
        public String toString() {
            return className + "#" + index;
        }
    }
}
```

### 권한

권한(capacity)을 문자열로 표현하는 경우도 있다.

예를 들어, 다음과 같이 스레드가 자신만의 변수를 저장해 사용할 수 있도록 하는 `ThreadLocal` 이라는 클래스를 설계한다고 해보자.

```JAVA
public class ThreadLocal\<T> {
    private ThreadLocal() {} //객체 생성 불가
    
    // 현 스레드의 값을 키로 구분해 저장한다.
    public static void set(String key, T value);
    
    // (키가 가리키는) 현 스레드의 값을 반환한다.
    public static T get(String key);
}
```

위 코드는 변수를 문자열 키로 저장하게 해두었다.

만약 스레드끼리 같은 key값을 가지게 된다면, 의도치 않게 같은 변수를 공유하게 되고, 그 결과 원하는 대로 기능하지 못하게 될 수 있다.<br>
또, 악의적인 클라이언트가 의도적으로 다른 이의 key 값을 집어넣으면, 해당 변수를 가져올 수 있게 된다.

이 문제는 문자열 key 대신, 위조할 수 없는 고유한 key, 즉 권한(capacity)을 사용하면 해결 된다.

아래 예제는 key를 별도의 클래스로 분리해 고유한 key를 생성했다.

```JAVA
public class ThreadLocal\<T> {
    private ThreadLocal() {} //객체 생성 불가

    public static class Key {
        key() {}
    }

    //위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
		    return new Key();
    }

    public static void set(Key key, T value);
    public static T get(Key key);
}
```

위와 같이 만들어도 괜찮겠지만, 좀 더 개선의 여지가 있다.

set / get 메서드는 더 이상 정적 메서드일 필요가 없다. 따라서 Key의 인스턴스 메서드로 변경한다. 이렇게 하면 Key는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다.

결과적으로 톱 레벨 클래스인 ThreadLocal 클래스는 별달리 하는 일이 없어지므로 중첩 클래스 Key를 ThreadLocal 자체로 변경할수도 있다.

```Java
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```

이렇게 하면 자바의 `java.lang.ThreadLocal`과 흡사한 구조를 가지게 된다.