# 아이템 07. 다 쓴 객체 참조를 해제하라

자바는 가비지 컬렉터가 힙 메모리를 관리해주며 다 쓴 객체의 메모리를 회수해 간다.<br>
하지만 자바를 사용한다고 메모리 관리를 소홀히 해서는 안된다. 

다음은 스택을 간단히 구현한 예제이다.

```JAVA
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

언뜻 보기에 특별한 문제는 없어보이지만, 이 스택을 오래 사용하다보면 **'메모리 누수'** 문제가 일어날 가능성이 있다.

#### 문제는 어디에?

문제는 바로 `pop()` 메서드에 있다. <br>`pop()`이 스택의 가장 마지막 원소를 반환할 때, 배열에서 해당 원소의 **참조를 없애주지 않고 반환하기 때문**이다.

예를 들어, 이 스택에 원소를 잔뜩 집어넣고, 또 잔뜩 빼준다고 하자. 우리는 이 스택이 가지고 있는 `elements` 배열이 빈 배열일 것이라 생각하지만, 실상은 다 쓴 참조(obsolete reference)를 그대로 가지고 있는 가득 찬 배열이 될 것이다. 이렇게 되면 배열이 원소들을 참조하고 있으니, 가비지 컬렉터가 메모리를 회수해줄 수가 없다. 

이 스택은 사용하면 사용할수록 가비지 컬렉션이 더 자주 일어나고 메모리 사용량이 늘어나면서 어플리케이션의 전체적인 성능의 저하를 초래할 것이다.

#### 해결은 어떻게?

메모리 회수가 안되는 이유는 다 쓴 참조를 가지고 있기 때문이니, **객체를 다 썼다고 판단되는 시점에 참조를 없애주면 된다**. 스택의 경우엔, "`pop()`으로 원소를 빼낼 때"가 될 것이다.

```JAVA
public class Stack {

    // ...

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
}
```
</details>

이처럼 사용 완료한 객체의 자리를 null로 대체해주면 간단히 참조를 없앨 수 있다.

이렇게 null 로 바꿔주면 생기는 이점이 하나 더 있다. 

다른 메서드를 이용해 size를 바꾸다가 실수로 잘못된 인덱스를 참조하게 돼서, 이상한 원소가 `pop()`을 통해 반환 됐다고 하자. 
* 만약 null 처리를 안했다면, 이 객체를 사용할 때 의도치 않게 정상 작동하는 것처럼 보일 것이다. 
* 하지만 null 처리를 했다면, 사용 즉시 NullPointerException을 던지고 종료하게 된다.

오작동의 범위가 작아야 버그를 찾는 것이 쉬우니, 이 편이 훨씬 좋을 것이다.

#### null 처리를 하면 안될 때와 해야할 때

그렇다고 무조건 참조를 null로 처리하는 것이 좋지만은 않다. 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효범위 밖으로 밀어내는 것이다. 유효 범위가 끝나 변수가 해제되면 객체 참조도 자동적으로 사라지기 때문이다. 만약 변수의 범위가 최소가 되게 정의했다면 이는 저절로 처리될 것이다.

참조를 null로 처리하는 것이 좋은 경우는 **클래스가 메모리를 직접 관리하는 경우**이다. 앞의 스택이 배열로써 원소가 들어갈 메모리를 직접 관리하는 경우 처럼, 논리적으로 더이상 참조가 일어나지 않을 곳의 객체들은, 그 자리에 대신 null을 넣어 참조를 없애는 것으로 가비지 컬렉터가 회수 대상인지 알 수 있게 해줘야 한다.

### 캐시에서의 메모리 누수

객체 참조를 캐시에 넣어놓고, 객체를 다 쓴 후에도 해제해주는 걸 잊어버려 참조가 남아있는 경우가 있다. 이럴 때는 다음과 같은 방법으로 해결 가능하다.

1. key가 없어지면 value도 없애는 방식<br>
    `WeakHashMap`은 해시맵의 엔트리 중 key가 참조되는 동안만 value가 남아있도록 구현되어있다.

2. 사용한지 오래된 entry를 없애는 방식<br>
    (ScheduledThreadPoolExecutor 같은) 백그라운드 스레드를 활용해 정기적으로 없애줄 수도 있고, 새 entry를 추가할 때 부수적으로 수행해 없애줄 수도 있다. `LinkedHashMap`은 entry를 추가할 때마다 `removeEldestEntry()` 를 수행해 오래된 entry를 없앤다.

### 리스너와 콜백에서의 메모리 누수

콜백을 등록만 하고 해지하지 않는다면, 콜백이 계속 쌓여 메모리 누수의 원인이 될 수 있다. 이는 앞서 말한 캐시 메모리 누수의 해결 방법과 동일하게, WeakHashMap에 저장함으로 가비지 컬렉터가 수거해가도록 할 수 있다.

# 추가 학습

## 자바에서 참조의 종류

Java에서는 효율적으로 GC를 동작시키기 위해, 참조(Reference)를 네 가지 유형으로 나누어 제거될 데이터에 우선 순위를 두었다.   

* 강한 참조 (Strong Reference)<br>
    `Integer prime = 1;`<br>
    위와 같은 가장 일반적인 참조 유형이다. prime 변수는 값이 1 인 Integer 객체에 대한 강한 참조를 가진다. 이 객체를 가리키는 강한 참조가 있는 객체는 GC대상이 되지않는다.
 
* 부드러운 참조 (Soft Reference)<br>
    `SoftReference<Integer> soft = new SoftReference<Integer>(prime);` <br>
    위와 같이 SoftReference Class를 이용하여 생성이 가능하다. 만약 `prime = null` 상태가 되어 더이상 원본(최초 생성 시점에 이용 대상이 되었던 Strong Reference)은 없고 대상을 참조하는 객체가 Soft Reference만 존재할 경우 GC대상으로 들어가도록 JVM은 동작한다. 다만 Weak Reference 와의 차이점은 메모리가 부족하지 않으면 굳이 GC하지 않는 점이다. 때문에 조금은 엄격하지 않은 Cache Library들에서 널리 사용되는 것으로 알려져있다.

* 약한 참조 (Weak Reference)<br>
    `WeakReference<Integer> soft = new WeakReference<Integer>(prime);`<br>
    위와 같이 WeakReference Class를 이용하여 생성이 가능하다. `prime = null` 이되면 (해당 객체를 가리키는 참조가 Weak Reference 뿐일 경우) GC 대상이 된다. 앞서 이야기 한 내용과 같이 Soft Reference와 차이점은 메모리가 부족하지 않더라도 GC 대상이 된다는 것이다. 다음 GC가 발생하는 시점에 무조건 없어진다.

* 팬텀 참조 (Phantom Reference)<br>
    `ReferenceQueue<Integer> referenceQueue = new ReferenceQueue<>();`<br>
    `PhantomReference<Integer> phantom = new PhantomReference<Integer>(Integer, referenceQueue));`<br>
    위와 같은 방식으로 생성 가능한 Phantom Reference는 GC에 가장 먼저 등록되는 대상이다. 작동 방식은 설명이 어려우니 [블로그 링크](https://luckydavekim.github.io/development/back-end/java/phantom-reference-in-java)로 대체하겠다.
