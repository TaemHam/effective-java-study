# 아이템 29. 이왕이면 제네릭 타입으로 만들라

- 클라이언트에서 직접 형변환을 해야하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
- 따라서 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 제네릭 타입으로 구현하자.

<br>

### 일반 클래스를 제네릭 클래스로 만들기

#### 클래스 선언에 타입 매개변수를 추가하기

- 다음 예제는 스택 클래스를 활용한 예제이다.
  
- 스택 클래스가 제네릭을 사용하도록 변경하려면 클래스 선언에 타입 매개변수를 추가하자.
  
```java
public class Stack {
    private Object[] elements;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ...
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
    }
    ...
}
```

```java
// 클래스 선언에 타입 매개변수를 추가하여 확장성을 높이는 동시에
// 인스턴스 선언 후 컴파일 오류를 사용할 수 있도록 타입을 제한할 수 있다.
public class Stack<E> {
    private E[] elements;

    public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
        }

    public void push(E e) {
        ...
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        E result = elements[--size];
    }
    ...
}
```
<br>

> **📌 배열을 사용하는 코드를 제네릭으로 만들 때 주의!**<br><br>
> - 제네릭 사용 시 E와 같은 실체화 불가 타입으로는 배열을 만들 수 없으며 2가지 방법을 사용할 수 있다.
> 1. 제네릭 배열 생성을 금지하는 제약을 우회하는 방법이 있다.
> 2. 배열을 사용하는 타입의 필드의 타입을 E가 아닌 Object로 변경하는 것이다.

### 배열을 제네릭으로 만드는 우회방법 1

- 제네릭 배열 생성을 금지하는 제약을 우회하는 방법

```java
public class Stack<E> {
    private E[] elements;
    
    // 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
    // 따라서 타입 안전성을 보장하지만,
    // 이 배열의 런타임 타입은 E[] 가 아닌 Object[]다!
    @SuppressWarnings("unckecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
}
```

- 위의 코드 중 elements 타입은 스택 클래스 안에서 private 이고 외부로 노출되는 일이 없다.

- 그러므로 Object 타입이라고 써 놓았으나 실제로는 E 타입의 클래스만 저장된다.

- 따라서 @SuppressWarnings를 통한 비검사 형변환은 안전하다.

<br>

### 배열을 제네릭으로 만드는 우회방법 2

- 배열을 사용하는 타입의 필드의 타입을 E가 아닌 Object로 변경하기

```java
public class Stack<E> {
    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        @SuppressWarnings("unckecked")
        E result = (E) elements[--size];

        elements[size] = null;

        return result;
    }
}
``` 

<br>

## 배열을 제네릭으로 만드는 2가지 방법의 비교

### 첫번 째 방법의 장점
- 가독성이 좋다.
- `private E[] elements` 를 사용하여 E 타입만 받는 배열이라고 명시한다.

### 두번 째 방법의 장점
- 첫번 째 방법과 달리 힙오염을 막을 수 있다. (힙 오염에 대한 자세한 내용은 아이템 32에서 다룬다.)

[힙 오염이란?](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TechnicalDetails.html#FAQ001)

<br>

## 모든 클래스에서 리스트를 사용할 수는 없다.
- 자바에서 리스트를 기본타입으로 제공하는 것이 아니기 때문에 ArrayList 같은 제네릭 타입도 결국 배열을 사용하여 구현해야 한다.
- HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.

<br>

## 한정적 타입 매개변수

- 타입 매개변수의 범위를 제한시켜 보다 제약조건을 강화한 타입 매개변수 선언 방식이다.


```java
// Delayed를 상속하는 타입만 타입 매개변수로 받을 수 있다.
class DelayQueue<E extends Delayed> implements BlockingQueue<E>

// Number를 상속하는 타입만 타입 매개변수로 받을 수 있다.
class Stack<E extends Number>
```