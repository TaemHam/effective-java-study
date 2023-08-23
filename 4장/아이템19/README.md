# 아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

- 아이템 18에서는 상속을 염두에 두지 않고 설계했고 상속할 때의 주의점도 문서화 해 놓지 않은 `외부`클래스를 상속할 때의 위험을 경고했다.
  
- 여기서 `외부`클래스란 프로그래머의 통제권 밖에 있어서 언제 어떻게 변경될 지 모른다는 뜻이다.

<br>

## 상속을 고려한 설계와 문서화란 정확히 무엇일까?
  
- 메서드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서로 남겨야 한다.
  - 즉 상속용 클래스는 재정의 할 수 있는 메서드들을 내부적으로 어떻게 사용하는지 문서로 남겨야 한다.
  
    > 📌 **AbstractMap을 참고하자.**
    > - AbstractMap는 상속용 클래스이다.
    > - AbstractMap의 각 메소드에는 메소드마다 문서가 잘 작성되어 있다.

    <br>

- 예를 들어 어떤 상황에 호출되는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 작성해야 한다.

  > 📌 **AbstractCollection의 remove 메소드를 참고하자.**
  > - remove 메소드는 iterator와 관련이 있으며, iterator의 remove 메소드를 구현하지 않으면 예외가 발생할 수 있다고 작성되어있다.
  > - remove 메소드 사용 시, 주의사항을 문서로써 작성해 두었으므로 좋은 예시라고 할 수 있다.

    <br>
  
## 훅(Hook)을 선별하여 protected 메서드 형태로 공개하는 방법도 있다.

- 내부 메커니즘을 문서로 남기는 것만이 상속을 위한 설계의 전부는 아니다.

- 효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅을 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.

- 예를 들어 AbstractList의 removeRange 메소드가 있다. clear 메소드에서는 removeRange를 훅 메소드로써 사용하고 있다.
  
  ```java
  // 외부로 공개되는 clear API는 내부의 훅 메소드를 호출한다.
  public void clear() {
    removeRange(0, size());
  }

  // List의 최종 사용자에게는 공개되지 않지만, 상속을 하는 하위 클래스에게
  // 최적화를 위한 로직을 작성할 수 있도록 protected로 열어주었다.
  protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);

    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
      it.next();
      it.remove();
    }
  }
  ```

  > 📌 **훅 메소드란?**
  > - 추상클래스에서 구현되는 메서드긴 하지만, 기본적인 내용만 구현되어 있거나 아무 코드도 들어있지 않은 메서드이다.
  > - 하위 클래스에서 훅 메소드를 구현하여 원하는 방향으로 로직을 구현할 수 있다.

  <br>

- 그러나 어떤 메서드를 protected로 열어줘야 하는지에 대한 기준은 없고, 직접 하위 클래스를 만들어봐야 알 수 있다.
  
- 널리 쓰일 클래스를 상속용으로 설계한다면 첫 배포 이후 수정이 아주 어렵다는 것을 인식하고, 반드시 배포 전에 하위 클래스를 만들어서 검증을 해야한다.

<br>

## 상속을 허용하는 클래스가 지켜야 할 제약들

### 1. 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능한 메서드를 호출해선 안된다.

- 자바에서는 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행된다.

- 만약 재정의한 메소드가 상위 클래스의 생성자에서 호출된다면 예상치 못한 오류를 만날 수 있다.

```java
public class Super {
  // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
  // 하위 클래스에서 재정의 가능 메서드를 재정의 했다면
  // 상위 클래스의 생성자에서도 하위 클래스에서 작성한 메소드가 호출되어 버린다.
  public Super() {
    overrideMe();
  }

  public void overrideMe() {

  }
}
public class Sub extends Super {

  private final Instant instant;

  Sub() {
    instant = Instant.now();
  }

  // 재정의 가능 메서드. 상위 클래스의 생성자가 호출한다.
  @Override
  public void overrideMe() {
    System.out.println(instant);
  }

  public static void main(String[] args) {
    Sub sub = new Sub();
    sub.overrideMe();
  }
}

```

  > 📌 **참고**
  > - private, final, static 메소드는 재정의가 불가능하므로 상위 클래스의 생성자에서 호출해도 된다.

<br>

### 2. Clonable나 Serializable 인터페이스를 구현한 클래스를 상속용으로 설계하지 말자.

- Clonable과 관련된 clone 메소드나 Serializable과 관련된 readObject 메소드 또한 새로운 객체를 만드는 생성자와 비슷한 효과를 낸다.

- 따라서 clone 메소드와 readObject 메소드 내부에서도 재정의 가능한 메소드를 호출하면 안된다.

- readObject의 경우 하위 클래스의 상태가 미처 역직렬화 (자바 객체로 만들어지는 경우)가 되기 전에 readObject가 호출되어 재정의 한 메소드부터 호출하게 된다.

- clone의 경우 하위 클래스의 clone 메소드가 복제본의 상태를 수정하기 전에 (복제본의 필드가 채워지기 전에) 재정의 한 메소드부터 호출하게 된다.
  
### 3. Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메소드를 가진다면 이 메서드들은 private가 아닌 protected로 선언해야 한다.

- 왜냐하면 이 메서드들은 private 로 선언한 경우 하위 클래스에서 무시되기 때문이다.

<br>

## 상속을 금지하는 2가지 방법

- 클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스에 안기는 제약도 상당하다.

- 따라서 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이 좋다.

- 클래스의 상속을 금지하는 2가지 방법은 다음과 같다.
  - 클래스를 final로 선언하기
  
  - 모든 생성자를 private나 package-private (default)로 선언하고 public 정적 팩터리 메서드 만들기

<br>

## 만약 그럼에도 일반 클래스를 상속해야 한다면..??

- 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이것을 문서로 남기는 것이다.

- 재정의 가능 메서드의 로직을 모두 private 메서드로 추출하고, 다른 곳에서는 private 메소들르 사용하게 한다.

```java

public class Sub extends Super {

  @Override
  public void add(int num1, int num2) {
    add(num1, num2);
  }

  // 재정의 가능 메소드의 로직을 private로 추출하면
  // 이 로직을 사용하고자 하는 다른 메소드에서
  // 재정의 가능 메소드가 아닌 private를 호출하면 되므로
  // 오류를 막을 수 있다.
  private void add(int num1, int num2) {
    add(num1, num2);
  }
}

```