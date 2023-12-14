# 아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

## 마커 인터페이스

마커 인터페이스(Marker Interface)란, 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 어떠한 속성을 가짐을 표시해주는 인터페이스이다.

대표적인 예로, `Serializable` 인터페이스가 있다. 해당 인터페이스는 단순히 자신을 구현한 클래스의 인스턴스는 `ObjectOutputStream`을 통해 쓸 수 있다고, 즉 직렬화할 수 있다고 알려주는 역할을 한다.

만약 Serializable을 구현하지 않았는데 ObjectOutputStream의 writeObject로 객체를 직렬화 하려고 한다면, 다음과 같은 오류가 발생한다.

```
java.io.NotSerializableException: org.test.TestObject

    at java.base/java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1185)
    at java.base/java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:349)
```

`writeObject0()` 메서드를 실제로 까보면, `else if (obj instanceof Serializable)` 로 타입 체크를 하는 것을 알 수 있다.

```JAVA
if (obj instanceof String) {
        writeString((String) obj, unshared);
    } else if (cl.isArray()) {
        writeArray(obj, desc, unshared);
    } else if (obj instanceof Enum) {
        writeEnum((Enum<?>) obj, desc, unshared);
    } else if (obj instanceof Serializable) {
        writeOrdinaryObject(obj, desc, unshared);
    } else {
        if (extendedDebugInfo) {
            throw new NotSerializableException(
                cl.getName() + "\n" + debugInfoStack.toString());
        } else {
            throw new NotSerializableException(cl.getName());
        }
    }
```

## 마커 애너테이션

위의 마커 인터페이스와 마찬가지로, 요소가 하나도 정의되지 않은 채로 어떠한 속성을 가짐을 표현한 애너테이션을 말한다. 아이템 40의 `@Override` 나 `@Deprecated`, `@Test`가 이것에 속한다.
```JAVA
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override { // 정의 된 게 아무것도 없다.
}
```

## 마커 애너테이션 vs 마커 인터페이스

두 방법 모두 같은 용도를 가지고 있는데 구분되는 이유가 뭘까?

* 마커 인터페이스의 장점

  1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 않다.<br>
    마커 인터페이스는 어엿한 타입이기 때문에, 마커 애너테이션을 사용했다면 런타임에야 발견될 오류를 컴파일타임에 잡을 수 있다.

  2. 마커 인터페이스는 적용 대상을 더 정밀하게 지정할 수 있다.<br>
    적용 대상을 @Target을 통해 Element.TYPE으로 선언한 애너테이션은 모든 타입(클래스, 인터페이스, 열거 타입, 애너테이션)에 달 수 있다.<br>
    즉, 부착할 수 있는 타입을 더 세밀하게 제한하지는 못한다.<br>
    특정 인터페이스를 구현한 클래스에만 적용하고 싶은 마커가 있는 경우, 마커 인터페이스를 사용하면 마킹하고 싶은 클래스에서만 그 인터페이스를 구현하면 된다. 그러면 마킹된 타입은 자동으로 그 인터페이스의 하위 타입임이 보장되는 것이다.

* 마커 애너테이션의 장점

1. 거대한 애너테이션 시스템의 지원을 받는다.

    애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 쓰는 쪽이 일관성을 지키는 데 유리하다.

이런 장단점을 지닌 방법들을 각각 어느 때에 사용하는 것이 좋을까?

* 마커 인터페이스 을 사용해야 하는 경우

    * 마킹된 객체를 매개변수로 받는 메서드를 작성할 일이 있는 경우<br>
    → 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일타임에 오류를 잡아낼 수 있다.

* 마커 애너테이션 을 사용해야 하는 경우

    * 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹해야 하는 경우<br>
    → 이 경우는 선택의 여지가 없다. 클래스와 인터페이스만이 인터페이스를 구현하거나 확장할 수 있기  때문이다.

    * 클래스와 인터페이스에 마킹해야 하더라도, 마킹된 객체를 매개변수로 받는 메서드를 작성할 일이 없는 경우

    * 애너테이션을 활발히 활용하는 프레임워크를 사용하는 경우