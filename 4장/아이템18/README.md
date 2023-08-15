# 아이템 18. 상속보다는 컴포지션을 사용하라

- 상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다. 그리고 잘못 사용하면 오류를 내기 쉬운  소프트웨어를 만들게 된다.
  
- 상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서라면 상속도 안전한 방법이다.
  
- 확장할 목적으로 설계되었고 문서화도 잘 된 클래스도 마찬가지로 안전하다.
  
- 하지만 일반적인 구체 클래스를 패키지 경계를 넘어, 즉 다른 패키지의 구체 클래스를 상속하는 일을 위험하다.


  > 📌 **여기서 말하는 상속은 구현 상속이다!**
  > - 클래스가 인터페이스를 `구현`하거나 인터페이스가 다른 인터페이스를 상속하는 것과는 다르다.

  <br>

  > 📌 **자바의 AbstractMap**
  > - 자바의 AbstractMap는 abstract 클래스로써 하위 클래스들은 AbstractMap의 기능을 상속받는다. 상속을 무조건 지양하는 것이 아니라 적절하게 사용하면 오히려 좋은 설계가 될 수 있다.

<br>

## 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.

- 다르게 말하면 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
  - 즉 상위 클래스가 수정되면 하위 클래스의 코드가 변하지 않더라도 하위 클래스가 오작동 할 수 있다는 의미이다.

- 이러한 이유로 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 발맞춰 수정되어야 한다.
 
- 이러한 클래스는 데이터 필드에 직접 접근할 수 있으니 `캡슐화`의 이점을 제공하지 못한다.

<br>

## 상속을 사용했을 때 단점

- 상속을 사용하는 경우에는 상위 클래스에 하위 클래스가 종속되어 버린다. (결합도가 높아짐)
 
- 자바에서는 1개의 클래스만 상속받을 수 있으므로 확장에 제한이 생길 수 있다.

### 아래는 HashSet의 기능을 재사용하고자 상속한 StringSet이다.

  ```java
  public class StringSet extends HashSet<String> {

    private int addCount = 0;

    @Override
    public boolean add(String s) {
      addCount++;
      return super.add(s);
    }

    @Override
    public boolean addAll(Collection<? extends String> c) {
      addCount += c.size();
      return super.addAll(c);
    }

    public int getAddCount() {
      return addCount;
    }
  }

	public static void main(String[] args) {
		StringSet set = new StringSet();

    // 이 경우 다음과 같은 프로세스로 메소드가 호출된다.
    // 1. StringSet의 addAll 메소드가 호출되면서 addCount가 리스트의 사이즈만큼 증가
    // 2. addAll 메소드 안에서 상위 클래스인 HashSet의 addAll을 호출하는데
    //    HashSet의 addAll 메소드에서는 add 메소드를 호출함
    // 3. 그런데 실제로 호출되는 add 메소드는 상위 클래스인 HashSet의 add 메소드가 아닌
    //    하위 클래스인 StringSet의 add 메소드가 호출된다.
    // 4. 따라서 addCount는 중복되어 증가하게 된다.
		set.addAll(List.of("글자 1", "글자 2"));

    // addCount는 2가 출력될거라 예상했지만 4가 출력된다!
		System.out.println(set.getAddCount());
	}

  ```

<br>

## 컴포지션을 사용했을 때 장점

- 컴포지션 된 클래스에 추가 메서드가 작성되어도 클라이언트 클래스는 영향 받지 않는다.
  - 그러나 컴포지션 된 클래스의 필드 타입이 바뀌거나 하는 경우에는 영향 받을 수 있다. (왜냐하면 클라이언트 클래스에서 컴포지션 된 클래스의 값에 접근 시 반환 타입이 변경되기 때문)

### 아래는 HashSet의 기능을 재사용하고자 HashSet을 컴포지션한 예제이다.
  
- 상속이 아닌 컴포지션을 통해 HashSet의 기능을 사용하자. (책의 예제는 너무 복잡하여 단순화 한 예제를 작성함)
  
- StringSet의 add 메서드나 addAll 메서드는 Set의 메소드를 호출하여 결과를 반환하는 역할을 하는데 이를 `전달 방식 (forwarding)`이라고 한다.

  ```java
  public class StringSet {

    private HashSet<String> set = new HashSet();
    private int addCount = 0;

    public boolean add(String s) {
      addCount++;
      return set.add(s);
    }

    public boolean addAll(Collection<? extends String> c) {
      addCount += c.size();
      return set.addAll(c);
    }

    public int getAddCount() {
      return addCount;
    }
  }

	public static void main(String[] args) {
		StringSet set = new StringSet();

		set.addAll(List.of("글자 1", "글자 2"));

    // addCount는 2가 출력된다.
		System.out.println(set.getAddCount());
	}

  ```

<br>

## Wrapper 클래스와 데코레이터 패턴

- 위의 컴포지션을 사용한 StringSet 클래스의 경우, Set의 기능을 사용하기 위해 컴포지션을 사용하며 Set 인스턴스를 감싸고 있다는 뜻에서 래퍼 클래스라고 한다.
  - 대표적인 예로 자바의 primitive 타입을 클래스로 활용할 수 있게 제공하는 Integer, Long 등이 있다.

- 또 다른 Set에 계측 기능 (addCount 기능)을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다.

  > 📌 **주의! 계측 기능을 덧붙이는 것이 데코레이터 패턴을 의미하는 것이 아니다.**
  > - 데코레이터 패턴은 특정 로직에 부가 기능을 덧붙이는 것을 의미한다.
  > - 책의 예제에서는 `계측 기능` 이라는 부가기능을 덧붙였을 뿐이다.
  > - 그리고 컴포지션을 활용해야만 데코레이터 패턴이 아니고, 데코레이터 패턴을 사용할 수 있는 방법은 여러가지가 있는데 책의 예제에서는 그 중 컴포지션을 활용한 것 뿐이므로 헷갈리지말자.

  [데코레이터 패턴이란?](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0_%ED%8C%A8%ED%84%B4)

<br>

## 그렇다면 상속은 쓰지 말아야 할까?

- 상속은 반드시 하위 클래스가 상위 클래스의 `진짜` 하위 타입인 상황에서만 쓰여아 한다.
  - 즉 is-a 인 관계에서만 사용해야 한다. (참고로 컴포지션은 has-a 관계라고 한다.)

  > 📌 **심지어 펭귄 (Penguin) 클래스는 Bird(새) 클래스를 상속받으면 안된다.**
  > - 왜냐하면 새는 날 수 있지만 펭귄은 날지 못하기 때문이다.
  > - 즉 상속을 사용하려면 진짜 is-a 관계인지를 고려해야한다는 것이다.

<br>

## 컴포지션 대신 상속을 사용하려고 할 때 마지막으로 다시 고려해야할 사항
- 확장하려는 클래스 (상위 클래스)의 API에 아무런 결함이 없는지?

- 결함이 있다면 이 결함이 하위 클래스까지 전파되어도 괜찮은지?
  - 하위 클래스에도 오류가 생기는 것을 의미한다.

- 상위 클래스가 확장을 고려해 설계되었는지?
  - is-a 관계라 하더라도 확장을 위한 상위 클래스가 아니라면 추후에 어떤 사이드 이펙트가 발생할지 예상할 수 없다.