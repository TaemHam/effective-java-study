# 아이템 79. 과도한 동기화는 피하라

## 동기화를 과하게 하면 안되는 이유

1. 성능을 떨어뜨림
2. 교착상태에 빠뜨림
3. 예측할 수 없는 동작을 낳기도 함

응답 불가(교착 상태)와 안전 실패(데이터 훼손)를 피하려면 **동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에게 양도하면 안된다**. 

예를 들어, 동기화된 영역 안에서는 재정의할 수 있는 메서드를 호출하면 안 되고, 클라이언트가 넘겨준 함수 객체를 호출해서도 안된다.<br>
동기화된 영역을 포함한 클래스 관점에서는 이런 메서드는 통제 불가능한 메서드로, 바깥 세상에서 온 외계인 메서드(Alien method)이다.

동기화 블록 안에서 외계인 메서드를 실행하면 어떻게 되는지, 구체적인 예시를 보자.

* **문제 코드**

다음 코드는 집합에 Pub/Sub 패턴을 적용시킨 `ObservableSet`이라는 클래스다.

```JAVA
// ForwardingSet은 Set을 컴포지션으로 받아, 받은 Set의 메서드를 그대로 사용하는 Set구현체이다. 
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    // 콜백 함수들을 저장시켜놓을 리스트
    private final List<SetObserver<E>> observers = new ArrayList<>();

    // 콜백 함수 저장
    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }

    // 콜백 함수 제거
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    // 저장된 콜백 함수들을 모두 호출
    private void notifyElementAdded(E element) {
        // 문제가 되는 부분!!
        // 동기화 블럭 내에서 외계인 메서드를 실행하고 있다!
        synchronized (observers) {
            for(SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }

    // Set에 원소 추가
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    // Set에 원소 여러개 추가
    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element);
        }
        return result;
    }
}
```

원소가 추가될 때마다, `observers`리스트에 저장시킨 `SetObserver`의 `.added()` 메서드를 실행하는 걸 볼 수 있다.

여기서 사용되는 `SetObserver`는 **함수형 인터페이스의 인스턴스**이다.

```JAVA
// BiConsumer<Observable<E>,E> 로도 대체할 수 있지만,
// 클래스명과 메서드명을 통해 용도를 직관적으로 표현하고, 
// 다중 콜백을 지원하도록 확장시킬 수 있게 하기 위해 사용되었다.
@FunctionalInterface
public interface SetObserver<E> {
	void added(ObservableSet<E> set, E element);
}
```

* **해피 패스**

이제 `ObservableSet`의 인스턴스를 만들고, <br>
추가되는 원소를 그대로 출력하는 콜백 함수를 `observers` 리스트에 저장한 후, <br>
다음과 같이 숫자 0 ~ 99를 이 `ObservableSet` 인스턴스에 집어 넣어보자.

```JAVA
public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver((set, element) -> System.out.println(element));

        for (int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}
```

위 코드는 0 ~ 99를 문제 없이 출력하고 종료된다.

* **에러 케이스 1**

하지만 콜백 함수로 다음과 같이, 특정 숫자가 추가되면 콜백 함수를 제거하는 콜백 함수를 넣으면 어떨까?

```JAVA 
        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if (element == 23)
                    set.removeObserver(this);
            }
        });
```

위 코드를 추가하면, 22 까지 실행되다가, 23에서 `ConcurrentModificationException` 예외를 던진다.<br>
관찰자의 `.added()` 메서드 호출이 일어나 `.removeObserver()` 메서드로 `observers` 리스트를 수정하려고 한 시점이, <br>
`.notifyElementAdded()`가 `observers` 리스트를 순회하는 도중이었기 때문이다.

* **에러 케이스 2**

같은 스레드에서는 제거가 불가능하다는 걸 알았다. 그럼 다른 스레드에게 제거를 부탁하면 어떻게 될까?

```JAVA
        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if (element == 23) {
                    ExecutorService exec = Executors.newSingleThreadExecutor();

                    try {
                        exec.submit(() -> set.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });
```

결론적으로 위 코드는 교착상태에 빠진다.

메인 스레드는 `.submit()` 으로 넘긴 일을 처리될 때까지 기다리는 `.get()` 메서드를 호출했는데,<br>
백그라운드 스레드는 `s.removeObserver()`를 호출하고 락을 획득하려고 시도하지만 <br>
이 락은 메인스레드가 이미 쥐고 있어 얻을 때까지 무한 대기를 하며 처리를 하지 못하기 때문이다.

만약 불변식이 깨져 백그라운드 스레드가 락을 획득하면 문제는 더 커진다.<br>
동시성 코드에 락이 없다면 발생하는 문제, 즉 데이터 훼손으로 이어질 수 있기 때문이다.

* **해결 방법 1**

이런 문제는 외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다. <br>
`.notifyElementAdded()` 메서드에서 리스트를 복사해서 사용하면 락 없이도 안전한 순회가 가능하다.

```JAVA
private void notifyElementAdded(E element) {
	List<SetObserver<E>> snapshot = null;
	synchronized(observers) {
		snapshot = new ArrayList<>(observers);
	}
	for (SetObserver<E> observer : snapshot)
		observer.added(this, element);
}
```

* **해결 방법 2**

메서드 호출을 동기화 바깥으로 옮기는 방법보다 괜찮은 방법이 있다. <br>
`CopyOnWirteArrayList`를 사용하는 것이다.

```JAVA
private final List<SetObserser<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

public void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers) {
         observers.added(this, element);
    }
}
```

이 클래스는 내부를 변경하는 작업에 대해 항상 복사본을 만들어 수행하기 때문에, <br>
내부 배열은 수정되지 않아 순회할 때 락이 필요없어 빠르다.

### 열린 호출 (Open call)

동기화 영역 바깥에서 호출되는 외계인 메서드를 열린 호출이라고 한다. <br>
외계인 메서드는 얼마나 오래 실행될지 알 수 없어서, 동기화 영역에서 호출된다면 성능 저하를 유발한다. <br>
열린 호출로 바꾼다면 실패 방지 효과 외에도 동시성 효율을 개선해준다. 

**기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것이다.**

멀티코어가 일반화된 요즘, 과도한 동기화가 초래하는 진짜 비용은 락을 획득하는데 얻는 비용이 아닌, **경쟁하느라 낭비하는 시간**이다. <br>
즉, 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용인 것이다. <br>
VM의 코드 최적화를 제한한다는 점도 과도한 동기화의 또다른 숨은 비용이다.

만약 동시성이 필요한 가변 클래스를 작성한다면, 다음 두 선택지 중 하나를 따르자.

1. 동기화를 전혀 하지 않고 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자. 
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자. <br>
    단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두 번째 방법을 선택해야 한다. 두번째 방법을 선택했다면 락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 사용할 수 있다.