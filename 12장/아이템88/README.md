# 아이템 88. readObject 메서드는 방어적으로 작성하라

## `.readObject()` 메서드

직렬화된 객체를 받아 역직렬화를 해주는 `.readObject()` 메서드는 Heap 영역에 없는 새로운 객체를 만들어내는 **생성자**와 역할이 비슷하다.

때문에 생성자에서 생길 법한 문제들은 `.readObject()` 메서드에서도 똑같이 일어나는데, <br>
따라서 생성자에서 지켜야 하는 다음 주의사항들을 동일하게 지켜야 한다. 

* 인수가 유효한지 검사해야 한다
* 매개변수를 방어적으로 복사해야 한다

만약 지켜지지 않으면, 공격자는 쉽게 해당 클래스의 불변식을 깨뜨릴 수 있다.

[아이템 50]에서 보았던, 시간적 기간을 표현하는 `Period` 클래스를 예시로 들어보자.

```JAVA
public final class Period implements Serializable {

   private Date start;
   private Date end;

   public Period(Date start, Date end) {
       this.start = new Date(start.getTime()); // 방어적 복사
       this.end = new Date(end.getTime());
       if (this.start.compareTo(this.end) > 0) { // 유효성 검사
           throw new IllegalArgumentException(start + " after " + end);
       }
   }

   public Date getStart() {
       return new Date(start.getTime());
   }

   public Date getEnd() {
       return new Date(end.getTime());
   }
}
```

이 클래스는 인스턴스가 만들어질 때 시작 일자가 종료 일자보다 늦지 않도록 검사하고 있다. <br>
또, 불변 클래스로 만들기 위해 각 일자를 조회할 때 Date의 복사본을 반환하고 있다.

하지만 `.readObject()` 메서드를 정의하지는 않아서, 자바의 기본 직렬화를 수행한다.

이러면 다음과 같은 상황들에서 문제가 생긴다.

1. 불변식이 깨진 직렬화 객체를 역직렬화 하는 경우

```JAVA
public class BogusPeriod {
    // 불변식을 깨뜨리도록 조작된 바이트 스트림
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
        0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
        0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
        0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
        0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
        0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
        0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
        0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
        0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
        (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
        0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
        0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
        0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
        0x00, 0x78
    };

    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p.start);
        System.out.println(p.end);
    }
    
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

이 프로그램을 실행하면 다음과 같이 불변식이 깨진 결과를 얻는다.

```JAVA
Fri Jan 01 12:00:00 PST 1999 // start 가 더 느리다.
Sun Jan 01 12:00:00 PST 1984 // end 가 더 이르다.
```

이를 해결하려면 `.readObject()` 메서드를 정의해 유효성 검사를 실시해야 한다.

```JAVA
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject(); // 기본 직렬화 수행
    if (start.compareTo(end) > 0) { // 유효성 검사
        throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
    }
}
```

2. private 필드로의 참조를 제공하는 경우

직렬화된 바이트 스트림 끝에 start와 end 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들 수 있다.

```JAVA
public class MutablePeriod {
    public final Period period;

    public final Date start;

    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);

            // 불변식을 유지하는 Period 를 직렬화.
            out.writeObject(new Period(new Date(), new Date()));

            /*
            * 이전 객체의 start, end 로의 참조를 추가.
            */
            byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 악의적인 참조
            bos.write(ref); // 시작 필드
            ref[4] = 4; // 악의적인 참조
            bos.write(ref); // 종료 필드

            // 역직렬화 과정에서 Period 객체의 Date 참조를 훔친다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }

    public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period mutablePeriod = mp.period; // 불변 객체로 생성한 Period
    Date pEnd = mp.end; // MutablePeriod 클래스의 end 필드
    
    pEnd.setYear(78); // MutablePeriod 의 end 를 바꿨는데
    System.out.println(mutablePeriod.end()); // Period 의 값이 바뀐다.
    }
}
```

위 코드를 실행하면 실제로 바뀌지 말아야할 종료 일자가 바뀌어 버린다.

```JAVA
Fri Apr 07 19:59:32 KST 1978
Mon Apr 07 19:59:32 KST 1969
```

이를 해결하려면 객체를 역직렬화할 때 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 방어적 복사하는 부분을 추가하면 된다.

```JAVA
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 방어적 복사를 통해 인스턴스의 필드값 초기화
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 유효성 검사
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +" after "+ end);
}
```

이 때, 방어적 복사를 유효성 검사보다 먼저 진행해야 하는데, <br>
반대라면 유효성 검사 이후 방어적 복사 이전에 불변식을 깨뜨릴 틈이 생기기 때문이다.

### 기본 직렬화 vs. 직접 정의

만약 `.readObject()` 메서드를 정의해야하는지 헷갈린다면, 다음 질문을 해보는 것도 좋다.

`transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?`

예 -> 기본 직렬화를 사용해도 좋다.
아니오 -> 직접 정의해 유효성 검사 + 방어적 복사를 수행 or 직렬화 프록시 패턴([아이템 90])을 사용