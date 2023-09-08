# 아이템 31. 한정적 화일드카드를 사용해 API 유연성을 높이라

> 📌 **매개변수화 타입은 `불공변(invariant)`이다.**<br>
  > - 즉 서로 다른 Type1, Type2가 있을 때 List\<Type1>과 List\<Type2>는 상속관계가 아니라는 뜻이다.
  > - 예를 들어 List\<Object>와 List\<String>은 상속관계가 아니며 이를 불공변이라고 한다.
  >     - Object와 String은 상속관계인데 List\<Object>와 List\<String>는 상속관계가 아니다.
  >     - 왜냐하면 List\<String>가 List\<Object>의 자식 클래스라고 한다면, 리스코프 치환 원칙에 어긋나기 때문이다.

  <br>

  > 📌 **와일드카드란?**<br>
  > - 특정한 패턴이 있는 문자열 혹은 파일을 찾거나, 긴 이름을 생략할 때 쓰이는 것이다.

  [와일드카드란?](https://ko.wikipedia.org/wiki/%EC%99%80%EC%9D%BC%EB%93%9C%EC%B9%B4%EB%93%9C_%EB%AC%B8%EC%9E%90)

<br>

## 불공변 방식보다 더 유연한 한정적 와일드카드 타입 사용하기

<br>

### Stack API 간단하게 알아보기

- 다음은 스택 클래스의 API를 간단하게 작성한 예제이다.
  
```java
static public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```
- 만약 일련의 원소를 스택에 넣는 pushAll 메서드와 popAll 메서드를 추가한다고 하자.

```java
// Iterable를 받아 스택에 요소를 추가한다.
public void pushAll(Iterable<E> src) {
    for(E e : src) {
        push(e);
    }
}

// Stack에서 Collection을 받아
// 스택의 요소를 Collection에 담아준다.
public void popAll(Collection<E> dst) {
    while(!usEmpty()) {
        dst.add(pop());
    }
}
```

- 만약 Stack<Number> 로 선언한 후 pushAll(intVal) 을 호출하면 오류 메세지가 나타난다.

- 우리는 Numebr의 하위 타입인 Integer를 제네릭으로 선언했을 때 Stack\<Integer>가 Stack\<Number>의 하위 타입일 것이라 기대한다.

- 그러나 제네릭을 사용하면 `불공변` 이기 때문에 Stack\<Integer>가 Stack\<Number>가 상속관계가 아닌 것이다.

<br>

### 한정적 와일드카드 사용하기

- 위의 pushAll 메서드의 파라미터는 `E 또는 E의 하위타입의 Iterable`이어야 한다.
  
- 위의 표현을 한정적 와일드카드로 표현하면 `Iterable\<? extends E>` 로 표현할 수 있다.

```java
public void pushAll(Iterable<? extends E> src) {
    for(E e : src) {
        push(e);
    }
}
```

<br>

```java
// 문제가 되는 코드
Stack<Number> stack = new Stack<>();
Collection<Object> objects = ...;

// 여기서 컴파일 오류가 발생한다.
// 왜냐하면 Collection<Object>과 Collection<E>는 상속관계가 아니기 때문이다.
stack.popAll(objects);
```
- 위의 popAll 메서드의 Collection 파라미터는 `E 또는 E의 상위타입을 타입으로 하는 Collection`이어야 한다.

- 위의 표현을 한정적 와일드카드로 표현하면 `Collection\<? super E>` 로 표현할 수 있다.

```java
public void popAll(Collection<? super E> dst) {
    while(!usEmpty()) {
        dst.add(pop());
    }
}
```

<br>

### 정리

- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하자.

  > 📌 **Pesc 공식**<br>
  > - producer-extends, consumer-super
  > - 즉 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하자.
  > - 만약 매개변수화 타입 T가 소비자라면 <? super T>를 사용하자.

<br>

- 생산자? 소비자?
  > 📌 **생산자와 소비자**<br>
  > - 매개변수 타입 T를 제네릭으로 가진 컬렉션이나 클래스를 기준으로 생각하자.
  >     - `Set<T> set`이 있다고 하면 Set을 기준으로 생각하는 것이다.
  > - 컬렉션 (또는 클래스) 인 Set에서 값을 꺼내서 어떤 처리를 한다면 Set은 생산자로 취급한다.
  >     - 그러니까 Set에서 값을 꺼내 다른 곳에 사용하는 것이므로 사용되는 대상은 T 또는 T의 하위타입이 할당될 수 밖에 없으므로 Set<? extends T> 로 확장할 수 있다.
  > - 컬렉션 (또는 클래스) 인 Set에 T의 값을 넣는다면 Set을 소비자로 취급한다.
  >     - 그러니까 Set에 T를 넣는 것이므로 Set은 T 또는 T의 상위 타입을 받을 수 있어야 하므로 Set<? super T> 로 확장할 수 있다.

[생산자와 소비자 알아보기](https://stackoverflow.com/questions/2723397/what-is-pecs-producer-extends-consumer-super)

<br>

### 실습

- 만약 두 개의 Set을 받아 union하는 메서드가 있다고 하자.

```java
// s1와 s2에서 값을 꺼내서 처리하므로 s1과 s2는 매개변수 타입 E의 생산자이다.
public static <E> Set<E> union(Set<E> s1, Set<E> s2)

// Pecs 공식에 따라 다음과 같이 확장할 수 있다.
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

- List 파라미터를 받아 최대값을 반환하는 메서드가 있다고 하자.

```java
// List의 요소를 탐색하면서 compareTo로 비교하며 최대값을 찾는다.
public <E extends Comparable<E>> E max(List<E> c) {
    if(c.isEmpty()) {
        throw new IllegalArgumentException("컬렉션이 비어있습니다.");
    }

    E result = null;

    for(E e : c) {
        if(result == null || e.compareTo(result) > 0) {
            result = Objects.requireNonNull(e);
        }
    }

    return result;
}

// List를 생각해보자.
// List c에서 요소를 "꺼내서" 처리하므로 List c는 생산자이므로 List<? extends E>로 확장할 수 있다.
// Comparable를 생각해보자. 
// Comparable에 요소를 "넣어서" compareTo를 통해 처리하므로 Comparable은 소비자라고 할 수 있다.
// 따라서 Comparable<? super E>로 확장할 수 있다.
public <E extends Comparable<? super E>> E max(List<? extends E> c) {
    if(c.isEmpty()) {
        throw new IllegalArgumentException("컬렉션이 비어있습니다.");
    }

    E result = null;

    for(E e : c) {
        if(result == null || e.compareTo(result) > 0) {
            result = Objects.requireNonNull(e);
        }
    }

    return result;
}
```

<br>

- max 메서드 활용 예시

- 먼저 예제에서 사용할 3가지 인터페이스의 관계를 알아보자.
- ScheduledFuture -> Delayed -> Comparable
  ```java
  public interface Comparable<E>
  public interface Delayed extends Comparable<Delayed>
  public interface ScheduledFuture extends Delayed, Future<V>
  ```

```java
// 한정적 와일드카드를 사용하지 않은 max 메서드
public <E extends Comparable<E>> E max(List<E> c) {
    ...
}

// 위의 메서드를 사용하면 다음 로직은 컴파일 오류가 발생한다.
// 왜냐하면 ScheduledFuture가 직접 Comparable을 구현한게 아닌
// 상위 클래스인 Delayed가 Comparable을 구현했기 때문이다.
// 따라서 ScheduledFuture는 Delayed와 compareTo 메서드를 통해 비교할 수 있다는 것이다.
// 그런데 max 메소드에서는 단순히 Comparable<E>로만 제한해 두었으므로 컴파일 오류가 발생하는 것이다.
List<ScheduledFuture<?>> scheduledFuture = ...
```

<br>


```java
// 한정적 와일드카드를 사용한 max 메서드
public <E extends Comparable<? super E>> E max(List<? extends E> c) {
    ...
}

// 위의 max 메서드에서는 Comparable<? super E>로 확장하였으므로 컴파일 오류가 발생하지 않는다.
List<ScheduledFuture<?>> scheduledFuture = ...
```

<br>

### 타입 매개변수와 와일드카드, 어느걸 쓰는 게 좋을까?

타입 매개변수와 와일드카드에는 공통된 부분이 있어, 둘 중 어느 것을 사용해도 괜찮을 때가 있다. 예를 들자면, 다음과 같이 메서드의 구현 코드에 제네릭 타입이 등장하지도 않는 상황이 있다.

```JAVA
public static <E> void printCollection(Collection<E> c) {
    for (Object e : c) {
        System.out.println(e);
    }
}
```

이 때, 타입 매개변수 \<E> 를 쓰는게 나을까? 와일드카드 \<?> 를 쓰는 게 나을까?
정답은 '와일드카드를 쓰는게 낫다'이다. 와일드카드는 제네릭이랑 달리 공변이므로, 와일드카드를 쓰면 메서드 구현에서 타입 매개변수를 신경쓸 필요가 없어지기 때문이다.

둘 중 어느 것을 사용해도 좋은지에 대한 판단은 다음의 간단한 규칙만 보면 된다.

> 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.

위의 `public static <E> void printCollection(Collection<E> c)` 에서는 Collection\<E>로 한 번만 등장했기 때문에 다음과 같이 와일드카드로 대체하면 된다.

```JAVA
public static <?> void printCollection(Collection<?> c) {
    for (Object e : c) {
        System.out.println(e);
    }
}
```

하지만 위 규칙을 무조건 따를 수 있는 건 아니다. 다음 예제를 보자.

```JAVA
// i 번째 요소와 j 번째 요소의 자리를 바꿔준다.
public static <?> void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

위 코드를 컴파일 하려고 하면, 다음과 같은 이상한 오류를 내뿜는다:
```
incompatible types: Object cannot be converted to CAP#1
```
원인은 List<?>에 null 외 어떤 타입도 집어넣을 수 없어 일어나는 현상이다. 이 때에는 똑같은 메서드인데 타입 매개변수를 \<E>로 사용하는 제네릭 메서드를 만들어 동작을 위임하는 방법이 있다.

```JAVA
public static <?> void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));    
}
```