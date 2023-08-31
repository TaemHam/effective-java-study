# 아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라

- 소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러에 에러가 발생하지는 않는다.

```java
// Utensil.java 파일에 작성된 Utensil 클래스와 Dessert 클래스

class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

<br>

> **📌 참고**<br><br>
> 그러나 다음과 같이 1개의 클래스 파일에 2개의 public 클래스가 작성되면 컴파일 오류가 발생한다.

```java
// 컴파일 오류가 발생한다!
public class Utensil {
    static final String NAME = "pot";
}

public class Dessert {
    static final String NAME = "pie";
}
```
- 만약 위의 Utensil.java 파일이 있다고 가정할 때 동일한 클래스를 담은 Dessert.java 클래스 파일을 작성했다고 하자.

```java
public class Main {

    public static void main(String[] args) {
        // IDE에서는 Utensil.java 파일과 Dessert.java 파일이
        // 별도의 패키지에 위치하면 컴파일 오류는 나지 않고
        // Main 클래스에서 어떤 클래스를 import 하느냐에 따라 
        // 아래 로직의 결과가 달라진다.

        // 직접 main 메소드가 위치한 클래스를 컴파일 할 때는
        // Utensil.java와 Dessert.java 중 어떤 파일을 먼저 컴파일 하느냐에 따라
        // 아래 로직의 결과가 달라질 수 있다.
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}

```

<br>

## 1개의 파일에 톱레벨 클래스를 여러 개 두지 말고 분리하자.

```java
// Utensil.java 파일
public class Utensil {
    static final String NAME = "pot";
}

// Dessert.java 파일
public class Dessert {
    static final String NAME = "pie";
}
```
> **📌 참고**<br><br>
> 톱레벨 클래스를 분리하면 유지보수성도 좋아지고, 예상치 못한 오류를 막을 수 있다.
