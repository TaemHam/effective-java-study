# 아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

- 자바 8 이전에는 기본 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가할 방법이 없었다.
  
- 자바 8 이후에는 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드를 도입했다.
  
    > 📌 **디폴트 메서드란?**
    > - 디폴트 메서드는 예전 버전의 인터페이스와 호환될 수 있으면서도 새로운 기능을 추가할 수 있는 것을 의미한다.

    > - 다음은 공식 문서에서 제공하는 디폴트 메서드의 정의이다.
    > - `Default methods enable you to add new functionality to the interfaces of your libraries and ensure binary compatibility with code written for older versions of those interfaces.`

<br>

## 디폴트 메서드를 그냥 사용하면 되는걸까?
  
- 디폴트 메서드를 선언하면 그 인터페이스를 구현한 후 디폴트 메서드를 재정의 하지 않은 모든 클래스에서 디폴트 메서드를 사용할 수 있다.
  
- 그러나 이 디폴트 메서드들이 모든 기존 구현체와 매끄럽게 연동되리라는 보장은 없다.

- 자바 8에서는 람다의 도입으로 인해 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가되었다.
  
- 대부분의 자바 라이브러리에 작성된 디폴트 메서드는 잘 작동하지만 `생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하는 것은 어렵다.`

- 예를 들어 apache common의 SynchronizedCollection가 있다.
  - SynchronizedCollection는 클라이언트가 파라미터로 제공한 객체를 쓰레드에 안전하도록 동기화하는 역할을 한다.
  - 그러나 특정 릴리즈에서는 removeIf가 구현되지 않았다. 그래서 상위 인터페이스의 removeIf를 사용할 수 밖에 없는데 그 removeIf 메서드는 `동기화` 로직이 작성되어 있지 않으므로 클래스가 제공하는 기능의 일관성을 깨뜨린다.
  
  > 📌 **책에 소개된 SynchronizedCollection에 관하여**
  > - 책에 소개된 SynchronizedCollection의 removeIf 메서드에 관한 내용은 2023.08.22 기준 4.4버전의 구현체에서는 removeIf를 구현하였다.

  <br>

- 다음과 같은 상황을 가정해보자.
  - MyInterface가 있었고, MyInterface를 구현한 MyImpl을 작성 후 배포하였다.
  - 배포 이후 MyImpl에 concat 메서드가 필요해서 추가했다. (concat 메서드는 로그를 찍고, 문자열을 합치는 기능을 한다.)
  - 그 이후 MyInterface를 구현한 모든 구현체에게 concat 메서드인 디폴트 메서드를 제공하려 했지만, 이미 concat이 구현되어 있는 MyImpl에는 디폴트 메서드가 적용되지 않는다. (예상치 못한 에러가 발생한다고 생각할 수 있다.)

  ```java
  public class Main {

    public static void main(String[] args) {
      MyImpl myInterface = new MyImpl();

      // MyInterface의 concat을 공통 메서도로 제공했지만
      // 이미 존재했던 MyImpl의 concat 메서드가 실행된다.
      String result = myInterface.concat("Hello ", "Java");

      System.out.println(result);
    }

    interface MyInterface {
      void print(String message);

      // MyImpl의 concat보다 나중에 작성된 후 배포된 디폴트 메소드이다.
      default String concat(String str1, String str2) {
        return str1.concat(str2);
      }
    }

    static class MyImpl implements MyInterface {

      @Override
      public void print(String message) {
        System.out.println(message);
      }

      // MyInterface의 concat보다 먼저 작성된 후 배포된 메소드이다.
      public String concat(String str1, String str2) {
        System.out.println("MyImpl 에서 실행된 concat 메서드");
        return str1.concat(str2);
      }
    }
  }

  ```

<br>

## 디폴트 메서드로 인한 예샹치 못한 에러를 처리하는 방법

- 디폴트 메서드로 인한 예샹치 못한 에러를 처리하는 방법은 구현체에서 재정의를 하는 것이다.
- 
- 디폴트 메서드를 재정의하고 필요한 로직을 추가 작성함으로써 상위 인터페이스의 디폴트 메서드만 호출했을 때의 예외를 처리할 수 있다.
  
  > 📌 **그러나 단점이..**
  > - 매 번 상위 인터페이스에 디폴트 메서드가 추가되었는지 확인해야 한다. (컴파일 오류조차 안나므로 일일이 확인해야 함)
  
  > - 디폴트 메서드는 인터페이스에서 제공하는 공통 로직의 성격이 강한데, 디폴트 메서드로 인해 에러가 발생하는 구현체마다 재정의를 해야하므로 번거롭다.

  <br>

## 그러니까 인터페이스를 설계할 때는 세심한 주의를 기울이자.

- 새로운 인터페이스라면 릴리즈 전에 반드시 여러 종류의 구현체를 만들어보고 인터페이스를 사용하는 클라이언트도 만들어봐야 한다.

- 인터페이스를 릴리즈 한 이후에도 결함을 수정하는 것이 가능한 경우도 있겠지만, 절대 그 가능성에 기대선 안된다.