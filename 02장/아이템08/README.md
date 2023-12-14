# 아이템 08. finalizer와 cleaner 사용을 피하라

### finalizer와 cleaner는 명시적으로 사용하지 말자!!
- 자바는 두 가지 객체 소멸자를 제공한다.
- Java 9부터 finalizer가 deprecated 되고 cleaner가 도입되었지만 cleaner도 사용하면 안된다.
- 왜냐하면 cleaner 또한 여전히 위험하고 느리고 예측 불가능하고 일반적으로 불필요하기 때문이다.

    > 📌 **자바의 finalizer와 C++의 destructor와 다른점?**
    >
    > - C++의 destructor는 특정 객체와 관련된 자원을 회수하는 보편적인 방법이다.
    >   - 그리고 비 메모리 자원을 회수하는데도 destructor가 쓰인다.

    > - 반면 자바에서는 가비지 컬렉터가 이 역할을 수행한다.
    >   - 자바는 try-with-resource와 try ~ finally를 사용해 해결한다.

    [비메모리 자원 알아보기](https://stackoverflow.com/questions/7037687/what-is-a-nonmemory-resource)


<br>

### finalizer와 cleaner를 사용하면 안되는 이유 1
- finalizer와 cleaner는 즉시 수행된다는 보장이 없다.

- 객체에 접근할 수 없게 된 후 finalizer나 cleaner가 실행될 떄 까지 얼마나 걸릴 수 알 수 없다.

- 예를 들어 파일 닫기를 finalizer나 cleaner에게 맡기면 시스템이 파일을 계속 열어두어 새로운 파일을 열지 못해 프로그램에 오류가 날 수도 있다.

- finalizer나 cleaner가 얼마나 빨리 수행될지는 GC의 알고리즘에 달렸고 이는 구현마다 천차만별이다.

<br>

### finalizer와 cleaner를 사용하면 안되는 이유 2
- 느리다.

- 특정 객체에서 finalizer를 호출하면 인스턴스의 자원 회수가 지연될 수 있다.

- 그렇게 되면 Out Of Memory 오류가 날 수 있다.

- finalizer 스레드는 다른 스레드보다 우선순위가 낮으므로 실행될 기회를 얻지 못할 수도 있다.

<br>

### finalizer와 cleaner를 사용하면 안되는 이유 3
- finalizer나 cleaner의 수행 여부를 보장하지 않는다.

- 그러므로 DB 같은 경우 공유 자원의 영구 락이 계속 쌓일 수 있다.

- finalizer나 cleaner 동작 수행 중 예외가 발생하면 예외가 무시되며 (스택 추적 내역 같은 것이 출력되거나 하지 않음) 처리할 작업이 남았더라도 수행되지 않고 그 순간 종료된다.
- 그나마 cleaner를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문에 이러한 문제가 발생하지 않는다.

<br>

### finalizer와 cleaner를 사용하면 안되는 이유 4
- finalizer나 cleaner는 심각한 성능 문제도 동반한다.

- finalizer나 cleaner를 명시적으로 호출하면 GC의 효율을 떨어뜨리므로 느려진다.

- 그러나 try-with-resource나 try ~ catch 방식으로만 사용하면 훨씬 빨라진다.

<br>

### finalizer와 cleaner를 사용하면 안되는 이유 5
- finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.

- finalizer 공격을 막으려면 클래스 자체를 final로 선언하거나, 아무 일도 하지 않는 finalizer 메서드를 만들고 final로 선언하면 된다. (상속을 못하도록 막는 작업이 필요하다.)

[finalizer 공격의 예시](https://yangbongsoo.tistory.com/8)

<br>

### finalizer와 cleaner의 대안은?
- 사용할 클래스의 AutoClosable을 구현해주고 클라이언트에서 close 메서드롤 호출하면 된다.

- 보다 효율적으로 사용하기 위해 자바의 try-with-resource 문법을 사용하자.

<br>

### 왜 finalizer와 cleaner를 사용하는 것일까?
- 사용자가 close 메서드를 호출하지 않는 것에 대비를 한 안전망이다.
    - close 메서드를 호출하지 않았을 때도 자원을 해제해야 하기 때문이다.

- 네이티브 피어와 연결된 객체를 자원해제 하고자 할 때 사용한다.
    - close 메서드를 호출하지 않았을 때도 자원을 해제해야 하기 때문이다.

    <br>

    > 📌 **자바 피어와 네이티브 피어란?**
    >
    > - 자바 피어란 네이티브 피어를 가지고 있는 자바 객체를 의미한다.
    > - 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 의미한다.
    
    <br>

- 네이티브 피어는 자바 객체가 아니므로 GC는 그 존재를 알지 못한다.
- 따라서 GC가 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못한다.
    - 그러므로 finalizer나 cleaner가 나서서 처리하지 적당한 작업이다.
    - 단 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때만 해당된다.

    - 그렇지 않으면 Autoclosable를 구현하여 close 메소드를 사용해야 한다.

<br>

### Cleaner 사용해보기
- Cleaner는 finalizer와 다르게 public API로 외부에 드러나지 않는다.

    ```java
    public class Room implements AutoCloseable {

        // 자바에서 제공하는 Cleaner 클래스를 정적 메서드를 통해 생성한다.
        private static final Cleaner cleaner = Cleaner.create();

        // 방의 상태. cleanable과 공유한다.
        private final State state;

        // cleanable 객체. 수거 대상이 되면 방을 청소한다.
        private final Cleaner.Cleanable cleanable;

        public Room(int numJunkPiles) {
            state = new State(numJunkPiles);
            // this에서 clean() 메소드를 호출하면
            // Runnable을 구현한 state가 호출된다.
            cleanable = cleaner.register(this, state);
        }

        @Override
        public void close() {
            // Room의 close 메소드가 호출 될 때
            // cleanable.clean() 메소드를 호출한다.
            // 그러면 cleanable에 등록한 state가 호출된다.
            cleanable.clean();
        }

        // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
        // Room을 참조하면 Room이 해제될 때 순환참조로 cleanble.clean()을 호출하기 때문이다.
        private static class State implements Runnable {
            int numJunkPiles; // 방 안의 쓰레기 수

            State(int numJunkPiles) {
                this.numJunkPiles = numJunkPiles;
            }

            // close 메서드나 cleaner가 호출한다.
            @Override
            public void run() {

                System.out.println("Cleaning room");
                numJunkPiles = 0;
            }
        }
    }
    ```

- try-with-resource를 사용한 올바른 예제

    ```java
    public static void main(String[] args) {
		try (Room room = new Room(7)) {
			System.out.println("안녕~");
		}
	}
    ```

- clean을 하지 않는 올바르지 않은 예제

    ```java
    public static void main(String[] args) {
        // room이 자원 해제 되지 않는 코드이다!
        // 언제 clean이 될지 예측할 수 없다.
		Room room = new Room(7);
        System.out.println("안녕~");
	}
    ```

### 결론
- finalizer나 cleaner는 웬만하면 쓰지 말자.
- 사용하는 경우에는 웬만하면 다음 경우에만 사용하자.
    - try-with-resource나 try ~ catch로 자원 해제되지 않았을 때를 대비한 안전망 역할을 할 때

    - GC가 자원을 해제 못하는 네이티브 자원을 회수할 때

- 만일 사용하더라도 불확실성과 성능 저하에 주의해야 한다.