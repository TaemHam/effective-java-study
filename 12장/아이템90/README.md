# 아이템 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

- Serializable을 사용하는 것은 생성자 이외의 방법으로 인스턴스를 생성하는 방법을 제공하는 것이다.

  - 그러므로 버그와 보안 문제가 일어날 가능성이 커진다는 뜻이다.

- 직렬화 프록시 패턴을 사용하면 이 문제를 줄일 수 있다.

<br>

## 직렬화 프록시 패턴이란?

- 직렬화 프록시 패턴은 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다.

  - 이 중첩 클래스가 바깥 클래스의 직렬화 프록시이다.

- 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다.

  ```java
  class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
      this.start = start;
      this.end = end;
    }

    // 이 메서드는 자바의 직렬화 시스템이 바깥 클래스의 인스턴스 대신
    // SerializationProxy 인스턴스를 반환하게 한다.
    private Object writeReplace() {
      return new SerializationProxy(this);
    }

    // 공격자가 불변식을 훼손하고자 여러 시도를 할 때
    // 다음과 같이 readObject 메서드를 작성하여 공격을 막을 수 있다.
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
      throw new InvalidObjectException("프록시가 필요합니다.");
    }

    // Peirod의 직렬화 프록시
    // 생성자에서 Peirod를 받아 값을 복사한다.
    private static class SerializationProxy implements Serializable {
      private final Date start;
      private final Date end;

      public SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
      }

      private static final long serialVersionUID = ...;

      // 역직렬화 시 호출되며, 직렬화 프록시를 바깥 클래스 인스턴스로 변환한다.
      private Object readResolve() {
        return new Period(start, end);
      }
    }
  }
  ```

- 위의 코드의 핵심은 readResolve 메서드이다.

- 직렬화는 생성자를 이용하지 않고도 인스턴스를 생성하는 기능을 제공하는데 직렬화 프록시 패턴을 사용하면 readResolve 메서드를 통해 `생성자`로 인스턴스를 만들게 한다.

  - 따라서 역직렬화된 인스턴스가 해당 클래스의 불변식을 만족하는지 검사할 또 다른 수단을 강구하지 않아도 된다.

<br>

## 직렬화 프록시 패턴의 장점

- 가짜 바이트 스트림 공격이나 내부 필드 탈취 공격을 프록시 수준에서 차단해준다.

  - 데이터만 가지고 자바 생성자를 통해 인스턴스를 만들기 때문이다.

- 어떤 필드가 직렬화 공격의 목표가 될지 고민하지 않아도 되며, 역직렬화 때 유효성 검사를 하지 않아도 된다.

- 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 동작한다.

  - 예를 들어 EnumSet의 경우가 있다.

  - EnumSet은 열거 타입의 원소가 64개 이하면 RegularEnumSet을 사용하고, 그보다 크면 JumboEnumSet을 사용한다.

  - 직렬화 프록시 패턴을 사용하면 RegularEnumSet을 직렬화 후 바이트 스트림에 데이터가 추가해도 JumboEnumSet으로 역직렬화 할 수 있다.

<br>

## 직렬화 프록시 패턴의 단점

- 클라이언트가 확장할 수 있는 클래스에 적용할 수 없다.

- 객체 그래프 순환이 있는 클래스에 적용할 수 없다.

- 성능이 느려질 수 있다.
