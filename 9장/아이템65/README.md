# 아이템 65. 리플렉션보다는 인터페이스를 사용하라

## 리플렉션 

리플렉션 기능(`java.lang.reflect`)을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다.

Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있다. <br>
또한 이 인스턴스들로는 그 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다.

나아가, Constructor, Method, Field 인스턴스를 이용해 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수도 있다.<br>
ex.) Method.invoke()는 어떤 클래스의 어떤 객체가 가진 어떤 메서드라도 호출할 수 있게 해준다.

### 리플렉션의 단점

하지만 리플렉션을 사용하면 따라오는 단점이 있다.

* 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다. -> 존재하지 않는 메서드를 호출하는 실수를 범하면, 런타임에서야 발견 가능.

* 코드가 지저분하고 장황해진다.

* 성능이 떨어진다. 리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다.

### 리플렉션 용법

일반적인 코드에서는 리플렉션이 필요 없다. 코드 분석 도구나 의존관계 주입 프레임워크와 같은 복잡한 애플리케이션에서나 리플렉션이 가끔 필요하다.

그래도 만약 리플렉션을 사용해야 한다면, 제한된 형태로 사용해야 단점을 피하고 이점을 취할 수 있다.

그럼 어떤 형태가 제한된 형태인가?

**인스턴스 생성시에만 리플렉션을 쓰고, 인스턴스 사용은 인터페이스를 통해서 하는 것**이다.

다음 코드는 args에 들어온 인수들 중, 첫 번째 인수는 집합 구현체 이름을 받아 불러오고, 나머지 인수들을 집합에 추가해 출력해주는 프로그램이다.

```JAVA
// 리플렉션으로 생성하고 인터페이스로 참조해 활용한다.
public static void main(String[] args) {
    
    // 클래스 이름을 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) Class.forName(args[0]); //비검사 형변환
    } catch (ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }
    
    // 생성자를 얻는다.
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }
     
    // 집합의 인스턴스를 만든다.
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스턴스화할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Set을 구현하지 않은 클래스입니다.");
    }
    
    // 생성한 집합을 사용한다.
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
}
```

위 코드는 단점이 두 가지 있다.

1. 리플렉션을 사용하지 않았다면 컴파일 타임에 잡을 수 있을 에러를, 리플렉션을 사용하기 때문에 런타임에 에러를 발생시키고, 그것도 6가지씩이나 발생시킨다.
2. 리플렉션을 사용하지 않았다면 생성자 호출을 한 줄로 쓸 수 있을 걸, 리플렉션을 사용하기 때문에 인스턴스 생성에 긴 코드를 작성해야 한다.

### 리플렉션을 이용한 의존성 관리

리플렉션은 런타임에 다른 클래스, 메소드, 필드에 접근하게 해준다.<br>
이 기능은 특히 런타임에 존재하지 않을 수 있는 코드와의 의존성을 관리해야 할 때 유용하다.

예를 들어, 프로그램이 특정 라이브러리의 최신 기능을 이용하려 할 때, 이 최신 기능이 오래된 버전에서는 존재하지 않을 수 있다.<br>
이런 경우, 일단 구버전으로 컴파일 한 후, 리플렉션을 이용하여 최신 기능이 존재하는지 **런타임에 확인**하고, 존재한다면 이를 이용하게 할 수 있다.

주의해야 할 점은 런타임에 해당 기능에 접근하는 것이 실패할 때 대체 방법을 준비해야 한다는 것이다.<br>
위의 그 예시에서 최신 기능을 찾는 데 실패했다면, 대체 수단을 이용하거나 기능을 줄여 동작하게 하는 등 조치를 취해야 한다.