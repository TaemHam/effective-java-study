# 아이템 09. try-finally 보다는 try-with-resources를 사용하라

자바 라이브러리에는 자원을 생성하면 `.close()` 메서드를 호출해 직접 닫아줘야 하는 자원들이 있다.

하지만 이런 자원 닫기 메서드는 클라이언트가 놓치기 쉽고, 이는 예측할 수 없는 성능 문제로 이어진다.

이런 자원들이 제대로 닫힘을 보장하기 위해, 전통적으로 `try-finally`가 쓰여왔다. 메서드가 정상적으로 종료되어 try 문이 끝나거나, 예외가 발생해 catch 문으로 넘어가더라도 finally 안에 작성된 코드의 실행은 보장할 수 있기 때문이다.

```JAVA
public class TryFinally {
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
}
```

하지만 `try-finally`문은 두 가지 문제점을 가지고 있다.

* try와 finally 둘 모두에서 예외가 발생하면, 앞의 예외는 사라진다.

    위의 firstLineOfFile 메서드를 예로 들어보겠다. 

    일단 기기에 물리적인 문제가 생겼다고 가정해보자.

    1. `br.readLine()` 메서드가 실패해 예외를 던지고, finally 문을 실행한다.
    2. `br.close()`도 같은 이유로 실패해 예외를 던지고 종료한다.

    이 경우 1에서 발생한 예외에 관한 정보는 스택 추적 내역에 남지 않게 된다. 이는 시스템에서의 디버깅을 어렵게 한다.

* 둘 이상의 자원을 사용한다면 코드가 복잡해 보인다.

    자원을 여럿 사용한다면, 사용하는 자원들을 모두 안전하게 닫기 위해 아래와 같은 중첩된 try-finally 문을 사용하게 된다.

    ``` JAVA
    public class TryFinally {

        private static final int BUFFER_SIZE = 0;

        static void copy(String src, String dst) throws IOException {
            InputStream in = new FileInputStream(src);
            try {
                OutputStream out = new FileOutputStream(dst);
                try {
                    byte[] buf = new byte[BUFFER_SIZE];
                    int n;
                    while ((n = in.read(buf)) >= 0) {
                        out.write(buf, 0, n);
                    }
                } finally {
                    out.close();
                }
            } finally {
                in.close();
            }
        }
    }
    ```

    딱 보면 알겠지만, inline이 많아져 가독성이 매우 좋지 않다.

#### 대안은 무엇이 있나?

이 문제들이 해결된 것이 자바 7에서 도입된 `try-with-resources` 문이다. 단어가 길어 당황스럽겠지만, 사실 try 블럭 시작 전에 자원을 생성하는 코드가 들어간 괄호를 추가한 것이 전부이다.

```JAVA
public class TryWithResource {
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }
}
```

자원을 여러 개 사용하더라도, 사용하는 갯수만큼 선언해주면 된다.

```JAVA
public class TryWithResource {
    private static final int BUFFER_SIZE = 0;

    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        }
    }
}
```

이를 사용하기 위한 조건은 단 한가지, **사용하는 자원이 AutoClosable 인터페이스를 상속해 `.close()` 메서드를 구현해 놓았다면 가능**하다.

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
