# 아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라.

## 키워드 짚고 넘어가기

### 프로세스

- 프로세스란 프로그램이 메모리에 올라가서 CPU를 할당받고 명령을 수행하고 있는 상태이다.

<br>

### 쓰레드

- 쓰레드란 프로세스가 운영체제로부터 할당 받은 자원을 이용하는 실행 단위 또는 흐름의 단위로써 프로세스는 반드시 하나 이상의 쓰레드를 갖는다.
- 1개의 프로세스 내에서 쓰레드는 각각 독립적인 stack을 할당받고, 여러 쓰레드는 힙 영역을 공유한다.

<br>

### 컨텍스트 스위치

- 1개의 CPU는 동일한 시간에 1개의 작업만 수행할 수 있다.

- 따라서 동시성으로 여러 개의 프로세스를 처리하려면 프로세스 간 전환하는 컨텍스트 스위치를 해야한다.

  - 쓰레드 단위의 컨텍스트 스위치도 있다.

- 컨텍스트 스위치가 일어나는 조건

  - 실행 중인 프로세스에서 IO 작업이 발생할 때 컨텍스트 스위치가 일어남

  - 운영체제의 스케줄러에 따른 스케줄링에 의해 컨텍스트 스위치가 일어남

<br>

### 쓰레드 간 공유하는 공유 데이터

- 각각의 쓰레드는 독립적인 stack 영역을 할당받는다. 그러나 자바에서 객체 등의 자원은 각 쓰레드의 stack이 아닌 힙 영역에 저장되기 때문에, 이 힙 영역에 저장되는 자원을 공유 데이터라고 한다.

<br>

### 동시성 문제

- 동시성 문제란 여러 쓰레드가 동시에 공유 데이터에 접근하면서, 원하지 않는 값이 도출되는 등의 정합성이 맞지 않는 문제를 말한다.

<br>

### 동기화

- 동기화란 프로세스 혹은 쓰레드 간 공유 영역에 대한 동시 접근으로 인해 발생하는 데이터 불일치를 막고, 데이터 일관성을 유지하기 위해 순차적으로 공유 영역에 접근하도록 보장하는 메커니즘이다.

- 간단히 말하면 동시성 문제를 해결하기 위한 기법을 동기화라고 할 수 있다.

<br>

### 원자적

- 모든 기계어 명령은 원자성을 가지며, 명령이 실행을 시작할 경우 그 명령의 수행 종료 시 까지는 인터럽트를 받지 않는 것을 의미한다.

  - 아주 간단하게 말해 연산이 끝날 때까지 방해받지 않고 연산을 마무리한다.

  ```java
  a = a + 1
  ```

  - 예를 들어 위의 코드가 있다고 하자.
  - 위의 코드는 총 3개의 연산으로 이루어진다.

    1. a 변수 꺼내오기 (로딩)
    2. a + 1 계산하기 (계산)
    3. a + 1의 결과를 저장하기 (적재)

  - 이 경우 각각의 연산은 원자적이지만, (1번 연산, 2번 연산) 등의 여러 개의 연산은 원자적이지 않으므로 중간에 컨텍스트 스위칭이 일어날 수 있다.

---

<br>

## 동기화를 위한 synchronized 키워드

- synchronized 키워드는 메서드나, 블록을 한번에 한 쓰레드씩 수행하도록 보장한다.

- 즉 자바에서 동시성 문제를 해결하기 위한 기법 중 하나이다.

- synchronized 키워드를 사용하면 다음과 같은 로직이 진행된다.

  - 한 객체가 일관된 상태를 가지고 생성된다.
  - 이 객체에 접근하는 메서드는 그 객체에 락을 건다.
  - 락을 건 메서드는 객체의 상태를 확인하고 필요시 수정한다.

- 즉 synchronized 키워드가 유효한 범위 내에서는 다른 쓰레드의 방해를 받지 않으며, 원자적으로 로직을 수행할 수 있다.

<br>

## synchronized의 중요한 기능 하나 더!

- 동기화 없이는 한 쓰레드가 만든 변화를 다른 쓰레드에서 확인하지 못할 수 있다.
- 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 쓰레드가 락의 보호하에 수행된 모든 로직의 최종 결과를 보게 해준다.

  > **📌 long과 double 외의 변수와 동시성**<br>
  >
  > - 자바 언어 명세상, long과 double을 제외한 변수를 읽거나 쓰는 동작은 원자적이다. (원서를 보면 or라고 적혀 있음)

- 그러나 `성능을 높이기 위해` 원자적 데이터를 읽고 쓸 때는 동기화 하지 말아야 겠다는 생각은 하면 안된다.

- 자바 언어 명세는 쓰레드가 필드를 읽을 때 항상 수정이 완전히 반영된 값을 얻는다고 보장하지만, 한 쓰레드가 저장한 값이 다른 쓰레드에게 보이는가는 보장하지 않는다. (가시성 문제)

- 따라서 동기화는 배타적 실행 뿐 아니라, 쓰레드 사이의 안정적인 통신에 꼭 필요하다.

<br>

## 공유 중인 원자적 데이터를 동기화하지 않았을 때 문제 (가시성 문제)

- 다른 쓰레드를 멈추는 작업을 생각해보자.

```java
private static boolean stopRequested;

public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
        int i = 0;
        while (!stopRequested) {
            i++;
        }
    });

    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);

    stopRequested = true;
}
```

- 위의 코드에서는 새로운 쓰레드를 시작 후, 1초를 쉬고 그 쓰레드를 종료시키고자 하는 로직이다.

- 그러나 이 코드를 실행하면 쓰레드가 종료되지 않고 무한히 실행될 수 있다.

- 왜냐하면 메인 쓰레드에서 변경한 stopRequested 값을 backgroundThread가 언제 보게될 지 보장할 수 없기 때문이다.

- 따라서 다음과 같은 코드로 변경해주어야 한다.

```java
private static boolean stopRequested;

// synchronized 메서드를 선언하여 동기화를 했다.
private static synchronized void requestStop() {
    stopRequested = true;
}

private static synchronized boolean getStopRequested() {
    return stopRequested;
}

public static void main(String[] args) throws InterruptedException {
    new Thread(() -> {
        int i = 0;
        while (!getStopRequested()) {
            i++;
        }
    }).start();

    TimeUnit.SECONDS.sleep(1);

    requestStop();
}
```

- 이렇게 멀티 쓰레딩의 가시성 문제를 해결할 수 있다.

- 또 다른 방법으로는, 자바에서 volatile 키워드를 사용하여 가시성 문제를 해결할 수 있다.

  [멀티 쓰레딩의 가시성](https://velog.io/@syleemk/Java-Concurrent-Programming-%EA%B0%80%EC%8B%9C%EC%84%B1%EA%B3%BC-%EC%9B%90%EC%9E%90%EC%84%B1)

<br>

<br>

## volatile 키워드를 사용한 가시성 문제 해결

- 다음과 같이 volatile 키워드를 사용하여 가시성 문제를 해결할 수 있다.

```java
private static volatile boolean stopRequested;

public static void main(String[] args) throws InterruptedException {
    new Thread(() -> {
        int i = 0;
        while (!stopRequested) {
            i++;
        }
    }).start();

    TimeUnit.SECONDS.sleep(1);

    stopRequested = true;
}
```

- 그러나 volatile 키워드를 사용한다고 하여 동기화가 된 것은 아니다.

- 예를 들어 아래와 같은 코드가 있다고 하자.

- volatile 키워드를 사용하여 가시성 문제는 해결할 수 있지만 여러 개의 쓰레드가 nextSerialNumber 변수에 접근한다고 하면 문제가 될 수 있다.

- 왜냐하면 증감 연산자가 있기 때문이다.

- 이를 해결하려면 volatile 키워드를 빼고 generateSerialNumber 메서드에 synchronized 키워드를 붙이자.

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

<br>

## 동시성을 위한 Atomic 클래스 사용하기

- java.util.concurrent.atomic 패키지에는 락 없이도 쓰레드에 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다.

- volatile 키워드는 동기화의 2가지 효과 중 통신 쪽 (가시성)만 지원하지만, atomic 클래스는 원자성까지 지원한다.

- 또 atomic 클래스는 성능도 우수하다.

<br>

## 동시성 문제를 피하는 방법

- 동시성 문제를 피하기 위해서는 애초에 가변 데이터를 공유하지 않는 것이다.

- 불변 데이터를 사용한다.

- 클래스 초기화 과정에서 객체를 정적 필드로 만들고, volatile, final 키워드를 사용하거나, 락을 통해 접근하는 필드에 저장한다.

- 동시성 컬렉션에 저장한다.

<br>

## 결론

- 동시성 문제를 피하기 위해 가변 데이터는 단일 쓰레드에서만 쓰도록 하자.

- 한 쓰레드가 데이터를 다 수정한 후, 다른 쓰레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화를 해도 된다.

- 그러면 그 객체를 다시 수정할 일이 생기기 전까지는 다른 쓰레드들은 동기화 없이 자유롭게 값을 읽어 갈 수 있다.

- 이런 객체를 `사실상 불변 (effectively immutable)`이라고 한다.

- 그리고 다른 쓰레드에 사실상 불변 객체를 건네는 행위를 `안전 발행 (safe publication)`이라고 한다.
