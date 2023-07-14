# 아이템 01. 생성자 대신 정적 팩터리 메서드를 고려하라.

### ❔ 정적 팩터리 메서드란?
  - public 생성자 대신, **정적 메서드로 해당 클래스의 인스턴스를 반환하는 메서드**

    ``` JAVA
    public class MyClass {
      // 생성자를 통한 인스턴스 생성
      public MyClass() {}
      
      // 정적 팩터리 메서드를 통한 인스턴스 생성
      public static MyClass getInstance() {
        return new MyClass();
      }
    }
    ```
    
  > **📌 참고**<br>
  > 여기서 정적 팩터리 메서드는 디자인 패턴의 팩터리 메서드와 다른 것이다!



### 👍 장점

  <p style="font-size: 18px;">
  1) 이름을 가질 수 있다.
  </p>

  정적 팩터리 메서드를 이용하면, **메서드 명에 생성되는 인스턴스에 대한 특성을 설명할 수 있다**. 

  아래와 같은 Account 클래스가 있다고 가정하자.

  ``` JAVA
  public class Account {

    private String name;
    private String role;

    public Account(String name, String role) {
      this.name = name;
      this.role = role;
    }
  }
  ```

  만약, **`role`을 인자로 주지 않았을 때 기본적으로 `USER`를 부여한 인스턴스를 생성하고자 한다**면, 다음 두 가지 방법으로 만들 수 있을 것이다.

  ```JAVA
    // 생성자를 이용한 방법
    public Account(String name) {
      this.name = name;
      this.role = "USER"
    }

    // 정적 팩터리 메서드를 이용한 방법
    public static Account createUserAccount(String name) {
      return new Account(name, "USER")
    }
  ```

  이제 다른 개발자가 위의 클래스를 이용한다고 하자.
  - 생성자를 이용한다면, 어떤 role이 부여되는지 코드를 보기 전까진 알 수가 없다.
  - 정적 팩터리 메서드는, 메서드 명으로 어떤 role이 부여되는지 설명 되어있다.


  <p style="font-size: 18px;">
  2) 호출 될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
  </p>

  정적 팩터리 메서드를 사용하면 인스턴스를 미리 만들어 놓고, 그 인스턴스를 다시 주는 방식으로 불필요한 객체 생성을 막을 수 있다.

  아래와 같이 생성자는 private으로 막고, 정적 팩터리 메서드인 `getInstance()` 메서드를 호출 해 미리 생성된 MyClass 인스턴스를 반환하도록 한다면, 객체를 싱글톤으로 만들 수 있다.

  ``` JAVA
  public class MyClass {
    
    private static final MyClass instance = new MyClass();

    private MyClass() {}
    
    public static MyClass getInstance() {
      return new MyClass();
    }
  }
  ```

  불변 클래스도 이 방식을 이용해 같은 값을 가진 객체는 같은 인스턴스임을 보장한다.

  > **📌 참고**<br>
  > 불변 클래스(Immutable Class)란, 인스턴스의 값이 변경되지 않는 클래스로, String, Boolean, Integer, Float, Long 등이 있다.

  생성 비용이 큰 객체가 자주 필요할 때 사용하면, 인스턴스를 생성하지 않고 줄 수 있으므로 성능 향상을 꾀할 수 있다. 플라이웨이트 패턴이 이 방식을 이용한다.

  > **📌 참고**<br>
  > 플라이웨이트(Flyweight) 패턴이란, 같은 속성을 갖는 객체들 사이에 가능한 한 많은 데이터를 서로 공유해서 메모리를 절약하는 패턴이다.


  <p style="font-size: 18px;">
  3) 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
  </p>

  생성자를 사용하면 생성되는 객체의 클래스가 하나로 고정된다. 하지만 정적 팩터리 메서드를 사용하면, **반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 가질 수 있다**. 이런 유연성을 응용하면 구현 클래스를 공개하지 않고, 객체를 반환할 수도 있다. 
  
  아래는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크의 예시이다.
  `NumberGenerator` 클래스의 `getRandomNumberGenerator` 메서드를 호출하면, 하위 타입인 난수 생성기를 얻고, `SequentialNumberGenerator` 메서드를 호출하면 또 다른 하위 타입인 순차 번호 생성기를 얻는 인터페이스이다.

  ```JAVA
  public interface NumberGenerator {

    static NumberGenerator getRandomNumberGenerator() {
      return new RandomNumberGenerator();
    }
    
    static NumberGenerator getSequentialNumberGenerator() {
      return new SequentialNumberGenerator();
    }

    public int generate();
  }

  class RandomNumberGenerator implements NumberGenerator {

    @Override
    public int generate() {
      int number = 난수 생성 알고리즘...;
      return number;
    }
  }

  class SequentialNumberGenerator implements NumberGenerator {

    private int number = 0;

    @Override
    public int generate() {
        return number++;
    }
  }
  ```

  사용자는 `NumberGenerator` 의 하위 타입인 `RandomNumberGenerator`, `SequentialNumberGenerator` 의 구현체를 직접 알 필요 없이, `NumberGenerator` 이라는 인터페이스를 사용하면 된다. 즉, API를 사용하기 위해 익혀야 하는 개념이 적어진 것이다.

  자바 컬렉션 프레임워크는 핵심 인터페이스들에 수정 불가, 동기화 등 기능을 더한 총 45개의 구현체를 제공하는데, 이 구현체들은 대부분 `java.util.Collections` 에서 정적 팩터리 메서드를 통해 얻도록 했다.

  ```JAVA
  // List 인터페이스로 구현 클래스의 인스턴스를 만들 수 있다.
  List<Integer> list = List.of(1, 2, 3, 4, 5);
  ```


  <p style="font-size: 18px;">
  4) 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
  </p>
  
  생성자는 해당 클래스의 인스턴스만 만들 수 있다. 하지만 정적 팩토리 메소드를 사용하면 같은 메소드라도 상황에 따라 다른 클래스 인스턴스를 반환할 수 있다. 적은 메모리를 사용해야하는 경우와 그 반대의 경우에 따라 다른 클래스 인스턴스를 반환함으로써 자원을 효율적으로 사용할 수 있다.

  다음은 특정 Enum 클래스에 대한 비어있는 Set을 만들어주는 `EnumSet`의 `noneOf` 메서드이다. `EnumSet` 클래스는 public 생성자 없이 정적 팩터리 메서드만 제공하는데, 원소의 수에 따라 64개 이하라면 `RegularEnumSet`, 초과라면 `JumboEnumSet`의 인스턴스를 반환한다.

  ```JAVA
  // EnumSet의 정적 팩토리 메소드는 경우에 따라 RegularEnumSet, JumboEnumSet 두개의 클래스의 인스턴스를 반환한다.
  public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
      throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
      return new RegularEnumSet<>(elementType, universe);
    else
      return new JumboEnumSet<>(elementType, universe);
  }
  ```

  사용자는 이 두 클래스의 존재를 몰라도 사용하는 데 지장이 없고, 또 다른 구현체가 추가되거나 기존 구현체가 삭제 되더라도 사용하는 데 지장이 없을 것이다.


  <p style="font-size: 18px;">
  5) 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
  </p>

  정적 팩터리 메서드를 사용하면 다른 클래스를 반환할 수 있는데, 반환할 객체의 인터페이스나 클래스 이름을 사용해 반환하도록 할 수 있다.

  ```JAVA
  // nextProviderClass 메소드 작성 시점에는 클래스가 존재하지 않아도 된다.
  // 클래스가 작성되면 그때 클래스의 이름을 전달하면 된다. 
  private Class<?> nextProviderClass() {
    ...
      return Class.forName(className);
      ...
  }
  ```

  `Class` 클래스의 `forName` 메서드는 인자로 주어진 클래스명을 찾아 메모리에 등록하는 정적 팩터리 메서드이다. 
  
  JDBC는 이 점을 이용해, 드라이버 클래스가 메모리에 올라가는 즉시 초기화 블럭을 통해 자신의 인스턴스가 `DriverManager`에 등록되도록 했다.

  ```JAVA
  public class Driver extends NonRegistereingDriver implements java.sql.Driver {
    static {
      try {
        java.sql.DriverManager.registerDriver(new Driver());
      } catch (SQLException E) {
        throw new RuntimeException("Can't register driver!");
      }
    }
  }
  ```
  
  이렇게 함으로써, 실제 드라이버 클래스에 대한 의존성 없이도 Connection을 생성할 수 있게 되었다.

  ```JAVA
  public static void main(String[] args) {
    String driverName = "com.mysql.jdbc.Driver";
    String url = "jdbc:mysql://localhost:3306/test";
    String user = "root";
    String password = "";
      
    try {
      // 클래스를 메모리에 올리는 즉시 초기화 블럭에 의해 DriverManager 에 드라이버 등록 
      Class.forName(driverName);

      // 드라이버로 어떤 것이 등록되어있는지 몰라도, 
      // 등록된 모든 드라이버를 순회하며 그 중 Connection이 만들어지는 것을 반환
      Connection connection = DriverManager.getConnection(url, user, password);

    } catch (ClassNotFoundException e) {
      e.printStackTrace();
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }
  ```



### 👎 단점


  <p style="font-size: 18px;">
  1) 상속을 하려면 public 이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
  </p>

  일반 생성자를 사용한 인스턴스 생성은 막고 정적 팩토리 메소드만을 사용하게 하려면 기존 생성자는 private으로 해야하고, 이렇게 되면 해당 클래스를 상속 할 수 없게 된다. 
  
  하지만 이 단점은 상속보다 컴포지션 사용을 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 장점으로 받아들일 수도 있다.

  > **📌 참고**<br>
  > 컴포지션(Composition)이란, 어떤 클래스A의 메서드를 다른 클래스B에서 사용하고자 할 때, 클래스A를 클래스B의 인스턴스 변수로 두는 것을 말한다. 
  > 이렇게 하면, 두 클래스는 결합도가 낮아져 코드의 수정을 최소화 하며 구성요소를 바꿀 수 있다.
  > <br>
  > 불변(Immutable) 타입으로 만든 객체는 값을 수정할 수 없어야 하는데, 상속이 열려있다면 하위 클래스에 setter를 만들어 불변성을 깰 수 있기 때문에, 상속하지 못하도록 해야한다.


  <p style="font-size: 18px;">
  2) 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
  </p>

  생성 방법이 `new 클래스(인자);`로 모두 같아 찾기 쉬운 public 생성자와는 달리, 정적 팩터리 메서드는 사용자가 이 메서드의 존재를 인지하고 있어야만 사용할 수 있다.

  이 문제를 완화하려면 API 문서를 잘 정리해놓는다거나, 널리 쓰이는 **메서드 명명 규칙을 사용해야 한다**.

## 정적 팩터리 메서드 명명 규칙

* **from** <br>
  매개 변수를 하나 받아 해당 타입 인스턴스를 반환 <br>
  `Date d = Date.from(instant);`

* **of** <br>
  여러 매개 변수를 받아 적합한 타입 인스턴스 반환 <br>
  `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`

* **valueOf** <br>
  from과 of 의 더 자세한 버전<br>
  `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE)`

* **instance** / **getInstance** <br>
  인스턴스를 반환, 같은 인스턴스임을 보장하지는 않음<br>
  `StackWalker luke = StackWalker.getInstance(options);`

* **create** / **newInstance** <br>
  인스턴스를 반환, 다른 인스턴스임을 보장<br>
  `Object newArray = Array.newInstance(classObject, arrayLen);`

* **get[타입]** <br>
  다른 타입의 인스턴스를 반환, 같은 인스턴스임을 보장하지는 않음<br>
  `FileStore fs = Files.getFileStore(path)`

* **new[타입]** <br>
  다른 타입의 인스턴스를 반환, 다른 인스턴스임을 보장<br>
  `BufferedReader br = Files.newBufferedReader(path)`

* **[타입]** <br>
  get타입 과 net타입의 간결한 버전<br>
  `List<Complaint> litany = Collections.list(legacyLitany)`

