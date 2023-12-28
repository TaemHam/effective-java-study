# 아이템 87. 커스텀 직렬화 형태를 고려해보라

만약 직렬화를위해 Serializable을 구현하기로 결정했다면, <br>
그 형태는 어떻게 할지 신중히 고민해야 한다.

개발 일정에 쫓기다 직렬화 형태를 나중에 설계하려고 미루고 기본적으로 제공하는 직렬화 형태를 사용한다면, <br>
기존에 만들어진 직렬화된 데이터때문에, 기존 형태를 버릴 수 없게 될 것이다. <br>
실제로 `BigInteger` 같은 자바 클래스도 이 문제를 겪고 있다. 

## 직렬화의 형태

먼저 기본적인 직렬화 형태가 왜 좋지 못한지 알아보자.

다음은 좋은 직렬화 형태를 만들기 위한 기본 규칙이다:

> **이상적인 직렬화 형태는 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.**

하지만 기본 직렬화 형태는 이 뿐만 아니라 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내고, 심지어 그 객체들이 연결된 위상까지 저장한다.

글로만 보면 이해하기 어렵기 때문에, 예시를 보자.

먼저 직렬화하기 좋은 클래스이다.

```JAVA
public class Name implements Serializable {
    
    /**
     * 성. null이 아니어야함
     * @serial
     */
    private final String lastName;
    
    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;
    
    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;
}
```

사람의 이름을 저장하는 이 클래스는, 논리적으로 보아도 [이름, 성, 미들네임] 세 개로 구성된다고 할 수 있다. <br>
따라서 이 클래스 형태 그대로 직렬화 해도 나쁘지 않다.

다음은 직렬화에 좋지 않은 클래스이다.

```JAVA
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
    
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    ... 생략
}
```

문자열 모음을 표현하는 이 클래스는, 논리적으로 보면 일련의 문자열을 표현하는 게 끝이지만, 물리적으로는 이중 연결 리스트로 연결되어 표현된다.

이 클래스에 기본 직렬화 형태를 사용하면, 연결 리스트를 구성하는 노드의 양방향 연결 정보를 포함해 모든 Entry 들이 저장될 것이다.

### 직렬화의 형태가 갖는 문제점

객체의 물리적 표현과 논리적 표현의 차이가 클 때, 기본 직렬화 형태를 사용하면 다음과 같은 문제들이 생긴다:

1. 공개 API가 현재의 내부 표현 방식에 영구히 묶인다. <br>
    앞에서 말했듯이, 아무리 다음 릴리스에서 직렬화의 형태를 바꾸려고 해도, 이전 방식을 처리할 수 있도록 남겨두어야 한다.

2. 너무 많은 공간을 차지할 수 있다. <br>
    내부 구현과 같이 물리적 구조까지 저장하게 되므로, 저장하거나 전송할 때 속도가 느리다.

3. 시간이 너무 많이 걸릴 수 있다. <br>
    직렬화 로직은 객체 그래프의 위상(어느게 먼저인지)에 관한 정보가 없으니 그래프를 직접 순회해볼 수밖에 없다.

4. 스택 오버플로우를 일으킬 수 있다. <br>
    직렬화를 위해 그래프를 순회할 때, 내부적으로는 재귀 순회를 하는데, 그래프의 크기가 크지 않더라도 스택 오버플로우를 일으키기도 한다.

위 `StringList` 예제로 마찬가지이다.

책에 따르면, 약 2000개의 `Entry`를 담는 것으로도, 스택 오버플로우가 일어난다고 한다.

이 문제를 해결하기 위해서는 `StringList` 가 가진 `Entry`의 모든 정보를 담는 것이 아닌, 각 `Entry`들의 논리적 관계를 해석해 그 의미를 담는 것이 좋다. <br>
이 경우에는, `Entry`의 갯수를 담고, `Entry`를 순서대로 나열하는 것으로 될 것이다.

```JAVA
public final class StringList implements Serializable {
    // transient는 직렬화에 포함시키지 않음을 표시한 키워드
    private transient int size = 0;
    private transient Entry head = null;
    
    // 이제는 직렬화되지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }
    
    ... 생략
    
    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     * 
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s) throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);
        
        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }
    
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();
        
        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for(int i = 0; i < numElements; i++) {
            add((String) s.readObject());
        }
    }
}
```

### transient 한정자

위의 예시를 보면 size와 head 필드에 `transient` 라는 키워드가 붙어있는 걸 볼 수 있다. <br>
`transient` 한정자는 직렬화와 관련된 키워드로, **`transient`가 선언된 인스턴스 필드는 직렬화 대상에 포함되지 않는다**. 

`transient` 한정자는 다음과 같은 상황에 사용된다:
* 다른 필드에서 유도되는 필드 (위의 size와 head가 여기에 속한다)
* JVM을 실행할 때마다 값이 달라져야하는 필드

주의할 점은, StringList 필드가 모두 transient 한정자가 붙어있더라도 `.defaultWriteObject()`를 호출해준다는 점이다. <br>
<details>
<summary>`.defaultWriteObject()` 메서드는 static 이나 transient 한정자가 붙지 않은 필드를 자동으로 직렬화에 포함시키는 메서드다.</summary>

![image](https://github.com/TaemHam/effective-java-study/assets/95671168/ee69b11f-29b7-46cb-b48d-8d5b2792dcc6)
</details>

하지만 이 클래스는 자동으로 직렬화 시킬 필드가 존재하지 않는데도 메서드를 호출했는데, <br>
그 이유는 향후 릴리스에 transient가 없는 필드가 추가될 경우, <br>
구버전에서 신버전 객체를 역직렬화 할 때 발생할 `StreamCorruptedException`을 피하기 위함이다.

### 멀티스레드 환경에서의 직렬화

만약 멀티스레드 환경에서 객체를 직렬화하게 될 경우, <br>
다음과 같이 직렬화 메서드에도 synchronized 키워드를 붙여줘야 한다.

```JAVA
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
}
```

생각해보면 당연하다. <br>
객체를 직렬화 하는 도중에 직렬화 하는 스레드가 멈추고 객체의 상태가 바뀐다면, 객체 일부는 이전 상태로 저장된 훼손된 객체 데이터가 담기게 되기 때문이다.

### 직렬 버전 UID

어떤 직렬화 형태를 사용하든 직렬 가능 클래스에 모두 직렬 버전 UID를 명시적으로 부여하자. <br>
이렇게 하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제가 사라진다. <br>
성능도 조금 빨라지는데, 직렬 버전 UID를 명시하지 않으면 런타임에 이 값을 생성하느라 복잡한 연산을 수행하기 때문이다.

```JAVA
private static final long serialVersionUID = <무작위로 고른 long 값>;
```

IntelliJ의 경우, `private static final long serialVersionUID` 까지 입력하고 Alt + Enter를 누르면 <br>
임의로 UID를 생성해주는 기능이 있다.

![image](https://github.com/KKS-Algorithm-Study/AlgoRepo/assets/95671168/b993e885-b489-4583-ba34-d7b39928b916)
