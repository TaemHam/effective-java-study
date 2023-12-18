# 아이템 81. wait과 notify 보다는 동시성 유틸리티를 애용하라

## wait과 notify & notifyAll이란?

- 다음과 같은 요구사항을 생각해보자.

  - 1개의 숫자를 2개의 쓰레드가 1씩 증가시킨다.

  - 2개의 쓰레드는 각각 다음과 같은 역할을 수행한다.

    - 홀수인 경우만 값을 증가 시키는 쓰레드

    - 짝수인 경우만 값을 증가 시키는 쓰레드

  - 각 쓰레드는 번갈아 가면서 값을 증가시켜야 한다.

- 위와 같은 요구사항에서 2개의 쓰레드는 서로 협력하며 작업을 진행해야 한다.

- 간단히 말하면 짝수 쓰레드는 값을 증가시킨 뒤, 홀수 쓰레드가 값을 증가시킬 때까지 기다리고, 자신의 차례가 되면 다시 값을 증가시키는 작업을 진행해야 한다.

<br>

### 2개의 쓰레드가 값을 교대로 증가시키는 로직 구현하기

<details>
<summary>상세 로직</summary>

```java
public class Counter {

    private static int UPPER_BOUND;
    private int count = 1;

    public void printOddNumber() {
        synchronized (this) {
            while (count < UPPER_BOUND) {
                while (count % 2 == 0) {
                    try {
                        wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                System.out.println(Thread.currentThread().getName() + "에서 >> " + count++);
                notify();
            }
        }
    }

    public void printEvenNumber() {
        synchronized (this) {
            while (count < UPPER_BOUND) {
                while (count % 2 == 1) {
                    try {
                        wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                System.out.println(Thread.currentThread().getName() + "에서 >> " + count++);
                notify();
            }
        }
    }

    public static void main(String[] args) {
        UPPER_BOUND = 100;

        Counter counter = new Counter();

        new Thread(() -> counter.printEvenNumber()).start();
        new Thread(() -> counter.printOddNumber()).start();
    }
}
```

</details>

<br>

[참고 링크](https://www.geeksforgeeks.org/print-even-and-odd-numbers-in-increasing-order-using-two-threads-in-java/)

<br>

### wait & notify를 사용하는 생산자, 소비자 패턴

- 생산자 역할을 하는 생산자 쓰레드와 소비자 역할을 하는 소비자 쓰레드가 있다.

- 두 개의 쓰레드는 SharedQueue 클래스의 인스턴스를 공유 데이터로 사용하여 값을 생산 및 소비한다.

- 생산자는 큐가 꽉 차면 소비자가 소비할 때까지 기다려야 한다.

- 소비자는 큐가 비어있으면 생산자사 생산할 때까지 기다려야 한다.

<details>
<summary>상세 로직</summary>

```java
class SharedQueue {

    // 별도의 락 변수 선언
    private final Object lock = new Object();

    private Queue<Integer> queue = new LinkedList<>();
    private int CAPACITY = 5;

    public void produce(int data) throws InterruptedException {
        synchronized (lock) {

            // 데이터 생산 시, 큐의 용량이 꽉 찼으면 데이터를 더 이상 생산하지 않고 대기
            while (queue.size() == CAPACITY) {
                System.out.println("큐가 가득 찼습니다. (생산 중지)");
                lock.wait();
            }

            queue.add(data);
            System.out.println("생산 : " + data);

            // 데이터를 생산했다면 잠자고 있을 다른 쓰레드를 깨움
            // wait이나 notifyAll은 같은 모니터 락을 획득하는 쓰레드끼리 협력한다.
            lock.notifyAll();
        }
    }

    public void consume() throws InterruptedException {
        synchronized (lock) {

            // 데이터 소비 시, 큐가 비어있으면 대기
            while (queue.isEmpty()) {
                System.out.println("큐가 비어 있습니다. (소비 중지)");
                lock.wait();
            }

            System.out.println("소비 : " + queue.poll());

            // 데이터를 소비했다면 잠자고 있을 다른 쓰레드를 깨움
            // wait이나 notifyAll은 같은 모니터 락을 획득하는 쓰레드끼리 협력한다.
            lock.notifyAll();
        }
    }
}

public class ProducerConsumerExample {

    public static void main(String[] args) {
        // 2개의 쓰레드가 sharedQueue 객체를 공유하여, 데이터를 생산하고 소비한다.
        // 서로 에러가 나지 않도록 특정 조건에서 다른 쓰레드를 기다린다. (wait, notify 사용)
        SharedQueue sharedQueue = new SharedQueue();

        Thread producer = new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                try {
                    sharedQueue.produce(i);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "생산자");

        Thread consumer = new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                try {
                    sharedQueue.consume();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "소비자");

        producer.start();
        consumer.start();
    }

}
```

</details>

---

<br>

## wait과 notify & notifyAll 보다 동시성 유틸리티를 사용하자.

- 이전에는 쓰레드 간 협력을 위해 wait과 notify & notifyAll을 직접 사용하였다.

- 그러나 고수준의 동시성 유틸리티가 도입되며 wait과 notify & notifyAll를 직접 사용하지 않고도 쓰레드 간 협력하는 코드를 작성할 수 있다.

- java.util.concurrent의 고수준 유틸리티는 3개의 범주로 나눌 수 있다.

  1. 실행자 프레임워크
  2. 동시성 컬렉션
  3. 동기화 장치

  > **📌 동시성 컬렉션**<br>
  >
  > - 동시성 컬렉션이란 기존 컬렉션에 동시성을 추가한 컬렉션이다.
  > - 내부에서 동기화 메커니즘이 작동하므로, 외부에서 락을 추가로 사용하면 속도가 느려진다.

    <br>

  > **📌 동기화 장치**<br>
  >
  > - 쓰레드가 다른 쓰레드를 기다릴 수 있게 하여, 서로 작업을 조율하고 협력할 수 있게 해준다.
  > - 대표적으로 CountDownLatch, Semaphore, CycleBarrier, Exchanger, Phaser가 있다.

<br>

## wait과 notify & notifyAll 보다 동시성 컬렉션을 사용하자.

<br>

### 동시성 컬렉션의 대표적인 예시 ConcurrentMap

- 동시성 컬렉션은 이미 내부에 구현된 동시성을 무력화하지 못하므로 (동시성을 무시 할 수 없으므로) 내가 원하는 여러 메서드를 묶어서 원자적으로 실행할 수 없다.

- 그래서 여러 동작을 1개의 원자적 동작으로 묶는 `상태 의존적 수정` 메서드들이 추가되었다.

> **📌 상태 의존적 수정 메서드란?**<br>
>
> - 특정 상태에 따라 처리하는 로직이 달라지는 메서드를 의미한다.
> - 대표적으로 Map의 putIfAbsent가 있다.

- 예를 들어 다음 로직을 수행한다고 하자.

  - key로 map에 값이 있는지 체크한다.

  - 값이 있다면 값을 기존 값을 유지하고, 값이 없다면 새로운 값을 넣어주자.

- 위의 로직을 구현하면 다음과 같다.

```java
Map<String, String> map = new ConcurrentHashMap<>();

String key = "java";
String value = "version 11"

if (map.get(key) == null) {
    map.put(key, value);
}
```

- 그러나 이 로직을 멀티 쓰레딩 환경에서 수행한다면, if문 이하 로직은 원자적이지 않다.

- 만약 putIfAbsent 메서드를 사용한다면 위의 로직을 원자적으로 수행할 수 있다.

- 이러한 메서드를 `상태 의존적 수정` 메서드라고 한다.

<br>

### 작업이 성공적으로 완료될 때까지 기다리는 컬렉션의 대표적인 예시 BlockingQueue

- 앞서 살펴봤던 wait과 notify & notifyAll 를 활용한 로직을 BlockingQueue를 활용하여 구현해보자.

- queue.add 대신 queue.put을 활용하면 queue에 여유가 있을 때까지 기다렸다가 값을 추가한다.

- queue.poll 대신 queue.take를 활용하면 queue에 데이터가 추가될 때까지 기다렸다가 값을 꺼낸다.

<details>
<summary>BlockingQueue를 활용한 생성자, 소비자 로직</summary>

```java
class SharedQueue {

    private int CAPACITY = 5;
    private BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(CAPACITY);

    public void produce(int data) throws InterruptedException {
        // put 메서드를 활용하여 데이터 생산 시
        // 큐의 용량이 꽉 찼으면 데이터를 더 이상 생산하지 않고 대기
        // queue.add(data); (이렇게 하면 오류남)
        queue.put(data);
        System.out.println("생산 : " + data);
    }

    public void consume() throws InterruptedException {
        // take 메서드를 활용하여 데이터 소비 시
        // 큐가 비어있으면 대기
        // System.out.println("소비 : " + queue.poll()); (이렇게 하면 오류남)
        System.out.println("소비 : " + queue.take());
    }
}

public class ProducerConsumerExample {

    public static void main(String[] args) {
        SharedQueue sharedQueue = new SharedQueue();

        Thread producer = new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                try {
                    sharedQueue.produce(i);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "생산자");

        Thread consumer = new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                try {
                    sharedQueue.consume();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "소비자");

        producer.start();
        consumer.start();
    }

}
```

</details>

<br>

## wait과 notify & notifyAll 보다 동기화 장치를 사용하자.

- 쓰레드가 다른 쓰레드를 기다릴 수 있게 하여, 서로 작업을 조율하고 협력할 수 있게 해준다.

- 대표적으로 CountDownLatch, Semaphore, CycleBarrier, Exchanger, Phaser가 있다.

- CountDownLatch는 하나 이상의 쓰레드가 또 다른 하나의 쓰레드의 작업이 끝날 때까지 기다리게 한다.

- CountDownLatch의 생성자는 int 값을 받으며, 이 값이 Latch의 countDown 메서드를 몇 번 호출해야 대기 중인 쓰레드를 깨우는지 결정한다.

<br>

### CountDownLatch를 활용한 쓰레드 별 작업 완료시간 측정하기

- 모든 작업이 완료되었을 때 시간을 측정하는 time 메서드를 만든다.

  - Executor 인터페이스를 활용하여 쓰레드 풀을 만든다.

  - 정수 값인 concurrency를 받아 쓰레드 갯수를 지정한다.

  - Runnalble 인터페이스를 활용하여 쓰레드가 작업할 상새 내용을 작성한다.

- time 메서드에서 주의할 점

  - 반드시 concurrency 값에 따라 CountDownLatch의 생성자와 쓰레드 갯수를 맞춰주어야 한다.

    - 왜냐하면 countDown 메서드가 지정된 횟수만큼 호출되어야 모든 쓰레드를 깨우기 때문이다.

    - 시간 간격을 잴 때는 System.currentTimeMillis가 아닌 System.nanoTime 을 쓰자.

    - 더 정확하고, 실시간 시계의 시간 보정에 영향받지 않는다.

<details>
<summary>상세 로직</summary>

```java
public class CountDownLatchExample {

    public static void main(String[] args) throws InterruptedException {
        Executor executor = Executors.newFixedThreadPool(10);
        int concurrency = 10;
        Runnable action = () -> {
            // 10초 이내로 쓰레드가 대기하는 시간을 랜덤으로 부여한다.
            try {
                ThreadLocalRandom random = ThreadLocalRandom.current();
                int waitTime = random.nextInt(10);

                Thread.sleep(waitTime * 500);

                System.out.println(Thread.currentThread().getName() + "의 대기 시간 >> " + waitTime);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        };

        long workTime = time(executor, concurrency, action);

        System.out.println("소요시간 >> " + ((double) workTime) / 1000000000);

        System.exit(0);
    }

    public static long time(Executor executor, int concurrency, Runnable action)
            throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        // executor에 각 쓰레드를 준비시킨다.
        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 쓰레드 별 타이머에 준비를 마쳤음을 알린다. (concurrency 수 만큼 호출된다)
                ready.countDown();

                try {
                    start.await();
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();
                }
            });
        }

        ready.await(); // 모든 작업자가 준비 될 때까지 기다린다. (countDown 메서드가 지정한 횟수만큼 호출 될 때까지)
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await(); // 모든 작업자가 일을 끝마치길 기다린다.

        return System.nanoTime() - startNanos;
    }

}
```

</details>

<br>

## wait과 notify & notifyAll를 사용할 때 주의점

- wait 메서드를 사용하는 표준 방식 (표준 템플릿)

- 아래처럼 wait 메서드를 사용할 때는 반드시 대기 반복문 (wait loop) 관용구를 사용하자.

- wait, notify, notifyAll은 synchronized 블록 밖에서 호출하면 안된다.

  - 왜냐하면 동기화를 위한 API이기 때문이다.

  - 특히 wait을 반복문 밖에서 절대 호출하지 말자.

```java
synchronized (lockObj) {
    while("조건이 충족되지 않음..") {
        lockObj.wait(); // 락을 해제하고, notify로 깨어나면 이후 로직을 수행한다.
    }

}
```

<br>

### 대기 조건 (wait loop의 조건)이 충족되지 않았을 때 동작을 이어가는 경우

- 때때로 wait loop의 조건이 충족되지 않았을 때 쓰레드가 동작을 이어가는 경우가 있다.

- 이는 락이 보호하는 불변식을 깨뜨릴 수 있으며, 다음과 같은 상황이 있다.

  - 쓰레드가 notify를 호출한 다음 대기 중이던 쓰레드가 깨어나는 사이에 다른 쓰레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.

  - 조건이 만족되지 않았음에도 다른 쓰레드가 실수로 혹은 악의적으로 notify를 호출한다.
    공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다.
    외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.

  - 깨우는 쓰레드는 지나치게 관대해서, 대기 중인 쓰레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 쓰레드를 깨울 수도 있다.

  - 대기 중인 쓰레드가 (드물게) notify 없이도 깨어나는 경우가 있다. 허위 각성(spurious wakeup)이라는 현상이다.

- 위의 4가지 경우 중, 마지막을 제외하고는 프로그래밍적 실수로 인해 락을 통한 일관성이 지켜지지 않는 경우를 의미하는 것 같다.

- notify와 notifyAll 중에서는 notifyAll을 사용하자.

  - 깨어나야 하는 모든 쓰레드가 깨어남을 보장하니, 항상 정확한 결과를 얻을 수 있다.

  - notify로 특정 쓰레드를 깨우는 것이 되지 않으므로 내가 원하는 쓰레드가 깨어나지 않을 수 있다.
