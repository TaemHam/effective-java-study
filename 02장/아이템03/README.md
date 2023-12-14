# 아이템 03. private 생성자나 열거 타입으로 싱글턴임을 보증하라

### ❔ 싱글턴이란?
  - 어플리케이션 실행 중 **인스턴스를 오직 하나만 생성할 수 있는 클래스**

#### 👍 장점
  - 인스턴스 생성 비용 감소
  - 메모리 낭비 방지
  - 객체가 여러 개 생성되면 위험한 경우 (Config 객체)

#### 👎 단점

  - private 생성자를 갖고 있어 상속이 불가능
  - 생성 방식이 제한적이기 때문에 Mock 객체로 대체하기가 힘들어 테스트가 어려움

### 자바로 싱글턴을 구현하는 여러가지 방법

  <p style="font-size: 22px;">
  1. public static final 변수로 접근 가능한 싱글턴
  </p>

  ``` JAVA
  public class Singleton {
    public static final Singleton INSTANCE = new Singleton();

    private Singleton() {}
  }
  ```

  - 장점
    - INSTANCE가 final 키워드를 가지고 있으므로, 클래스가 싱글턴임이 API에 명백히 드러남.
    - 매우 간결함
  
  - 단점 
    - 인스턴스 생성 비용이 크다면, 애플리케이션 로드가 오래걸릴 수 있음
    - 리플렉션 API의 `AccessibleObject.setAccessible`로 private 생성자 호출 가능
      - INSTANCE가 존재하는데 생성자가 호출되면 예외를 반환하도록 해 해결 가능
    - Serializable을 상속 받았다면, 역직렬화시 새 객체 생성 
      - `private Object readResolve()` 메서드에서 INSTANCE를 반환하도록 해 해결 가능

  <p style="font-size: 22px;">
  2. getInstance()와 같은 정적 팩토리 메서드로 접근 가능한 싱글턴
  </p>

  ```JAVA
  public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
      return INSTANCE;
    }
  }
  ```
  - 장점
    - API 변경 없이 싱글턴 여부를 바꿀 수 있음
    - 정적 팩터리 메서드를 `제네릭 싱글턴 팩터리`로 변환 가능
    - 정적 팩터리 메서드의 메서드 참조를 공급자(Supplier)로 사용 가능
  - 단점
    - 위의 변수 접근 싱글턴과 동일한 단점을 가짐

  > **📌 참고**<br>
  > 제네릭 싱글턴 패턴이란, 제네릭으로 타입설정 가능한 싱글턴 인스턴스를 만들어두고, 반환 시에 제네릭으로 받은 타입을 이용해 타입을 결정하는 것이다.
  > ``` JAVA
  >  public class GenericFactoryMethod {
  >    public static final Set EMPTY_SET = new HashSet();
  >
  >    public static final <T> Set<T> emptySet() {
  >       return (Set<T>) EMPTY_SET;
  >    }
  >  }
  > ```
  > 위와 같은 코드에서 `GenericFactoryMethod.emptySet()`를 `Set<String>` 으로 받아 String을 넣을 수 있고, 후에 같은 Set을 `Set<Integer>`로 받아 String이 들어간 Set에 Integer를 넣을 수도 있다. 


  > **📌 참고**<br>
  > 정적 팩터리 메서드의 메서드 참조를 공급자로 사용하는 예시 
  > ``` JAVA
  >  public class SupplierUser {
  >    public void useSupplier(Supplier<Singleton> singletonSupplier) {
  >       Singleton singleton = singletonSupplier.get();
  >       singleton.doSomething();
  >    }
  >
  >  public static void main(String[] args) {
  >    SupplierUser supplierUser = new SupplierUser();
  >    supplierUser.useSupplier(Singleton::getInstance);
  >    }
  >  }
  > ```

  <p style="font-size: 22px;">
  3. 열거 타입으로 생성하는 싱글턴
  </p>

  ```JAVA
  public enum Singleton {
    INSTANCE;
  }
  ```
  
  - 장점
    - public 필드 방식과 비슷하지만 더욱 간결함
    - 추가적인 노력 없이 직렬화 가능
    - 리플렉션 공격에도 자유로움
    - 인터페이스 구현이 가능해 테스트 코드 작성도 가능
  - 단점
    - 다른 클래스를 상속할 수 없음