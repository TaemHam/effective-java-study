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

try-with-resources를 사용하면 다음과 같은 장점을 지니게 된다.

1. 예외가 사라지지 않고 숨겨진다.

    앞의 상황처럼 `.readLine()`과 `.close()` 둘 모두에서 예외가 발생한다면, close에서 발생한 예외는 숨겨지고 readLine 에서 발생한 예외가 출력된다. 숨겨진 예외는 없어지는 것이 아닌, 'suppressed' 상태로 존재하게 되는데, 이는 `.getSuppressed()` 메서드를 통해 접근할 수 있게 된다.

2. 가독성이 좋아진다.

    finally 절이 사라진다는 점, 그리고 여러 자원이 하나의 inline 레벨에서 관리된다는 점에서 가독성이 더욱 뛰어나졌다. 특히 예외 처리로 catch 문을 사용할 때, 각 자원들마다 발생할 수 있는 예외가 다를 수 있어, catch 문의 inline 레벨도 각각 달라야 했는데, try-with-resources 에서는 모두 같은 레벨에서 처리가 가능하다.

# 추가 학습

## try-with-resources를 사용할 때 예외 발생 시 어떻게 처리되는가?

- try의 () 문 안에서 예외가 발생한 경우
  - 메모리에 할당되지 않고 null이 되어버리므로 close를 호출할 필요가 없으니 AutoCloseable의 close 메소드도 호출되지 않는다.
  
- try의 () 문 안에서 여러 개의 인스턴스를 생성하려고 하는데, 특정 클래스에서 생성 시점에 예외가 발생 시 이미 생성된 인스턴스에 대해서는 close가 호출되어 자원 해제를 한다.

    ```JAVA
    class AutoCloseableCloseTest {

        @Test
        void closeTest1() throws Exception{
            try (
                // 애초에 mySocket 변수에 인스턴스가 할당되지 않으므로
                // AutoClosable의 close 메소드를 호출할 필요가 없다.
                MySocket mySocket = new MySocket("");
                ){

            }
        }

        @Test
        void closeTest2() throws Exception{
            try (
                // mySocket1이 할당 된 후, mySocket2를 생성할 때 예외가 발생하였다.
                // 이 경우 mySocket1의 close 메소드는 호출이 되며,
                // mySocket2의 close 메소드는 호출이 되지 않는다.
                MySocket mySocket1 = new MySocket("1번 소켓");
                MySocket mySocket2 = new MySocket("");
                MySocket mySocket3 = new MySocket("3번 소켓");
                ){

            }
        }

        @Test
        void closeTest3() throws Exception{
            try (
                // 정상적으로 자원 할당 후 try 블록 안에서 예외 발생 시
                // close 메소드가 호출된다.
                MySocket mySocket = new MySocket("소켓");
                ){
                    // myMethod 호출 시 예외가 발생한다.
                    mySocket.myMethod();
            }
        }

        static class MySocket implements AutoCloseable{
            private final String param;

            public MySocket(String param) {
                if(param.equals("")) {
                System.out.println("예외 호출");
                    throw new IllegalArgumentException();
                }
                this.param = param;
            }

            public void myMethod() {
                throw new IllegalArgumentException();
            }

            @Override
            public void close() throws Exception {
                System.out.println("close 호출");
                System.out.println("param = " + param);

            }
        }
    }
    ```