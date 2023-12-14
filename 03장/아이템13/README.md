# 아이템 13. clone 재정의는 주의해서 진행하라

### Cloneable 이란?

어떤 객체의 복사를 허용할 때 구현하는 인터페이스로, 구현할 메서드가 하나도 없는 마커 인터페이스 중 하나이다. 복사할 때 사용되는 clone 메서드는 Object 클래스에 있기 때문에, Cloneable을 구현하지 않아도 clone 메서드가 호출 가능하나, 이 경우 CloneNotSupportedException이라는 오류룰 던진다. 

### Clonable의 문제점

`protected native Object clone() throws CloneNotSupportedException;`

* clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이다. <br>
    보통은 구현할 인터페이스에 메서드가 있을 것이라 생각하지만, Cloneable은 그렇지 않음. Cloneable은 그저 구현된 클래스에서 호출된 clone 메서드가 `CloneNotSupportedException`를 던지지 않도록 바꿔주기만 함.
* clone 메서드가 protected로 정의되어있다. <br>
    인터페이스를 `implements` 한 것 만으로는 사용 불가능, 재정의 해줘야 함.
* 상위 클래스의 clone을 호출해 복사해야한다. <br>
    clone을 재정의할 클래스의 상위 클래스는 모두 Cloneable을 상속해 clone을 재정의해야 함.
* checked exception으로 CloneNotSupportedException가 발생하도록 정의되었다. <br>
    clone을 처음 재정의할 때(어떤 것도 extends 하지 않은 최상위 클래스에서)만 처리해야 하는데, 이미 Cloneable을 구현함으로 발생할 일이 전혀 없게 되었으므로, checked exception일 필요가 없음. 오히려 try-catch 문을 붙여줘야 해서 가독성이 떨어짐. 
* clone 메서드의 일반 규약이 존재하나, 강제성이 없다.<br>
    상위 클래스의 clone 메서드가 원하는대로 구현되어있지 않을 수 있음. <br>
    예를 들어, ParentClass 의 clone 메서드가 `super.clone()`으로 복사한 게 아닌 `new ParentClass();` 처럼 생성자로 생성한 **새로운 인스턴스를 반환**하면, ParentClass를 상속한 ChildClass 의 복사본은 ParentClass 타입을 가지게 되어버림. 하지만 규약에 강제성이 없어 잘못된 코드는 아님.

    <details>
    <summary>clone 메서드 규약 보기</summary>

    >이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
    >
    >x.clone() != x (주솟값이 다르다.)
    >
    >또한 다음 식도 참이다.
    >
    >x.clone().getClass() == x.getClass() (타입이 동일하다.)
    >
    >하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다. 한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.
    >
    >x.clone().equals(x) (논리적으로 동치이다. 즉, 핵심 필드가 동일하다.)
    >
    >관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
    >
    >x.clone().getClass() == x.getClass()
    >
    >관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.
    </details>
    

### 그럼에도 Cloneable을 사용하는 이유?

Cloneable 로 객체를 복사하는 방식이 널리 쓰이기 때문이다.

### Cloneable을 안전하게 사용하는 방법

다음은 일반적으로 clone 메서드를 구현한 코드이다.

```JAVA
@Override public MyClass clone() {
    try {
        MyClass result = (MyClass) super.clone();
        return result;
    } catch (ClassNotSupportedException e){
        throw new AssertionError();
    }
}
```

1. 가변 객체를 참조할 경우

클래스의 인스턴스 필드로 배열 같은 가변 객체가 존재할 경우, super.clone()으로는 완전한 복사를 할 수 없다.

지난 아이템에 사용했던, 배열을 이용해 스택을 구현한 코드를 보자.

```JAVA
public class Stack extends Cloneable{
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
 
    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
 
    public void push (Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
 
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = element[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
    if (elements.length == size)
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

이 스택 클래스에 위의 clone 메서드를 붙여주면, 복사된 Stack 인스턴스는 올바른 size 값을 가지지만, elements 배열은 **원본 Stack 인스턴스와 똑같은 배열을 참조하게 된다**. 둘 중 하나에서 push 하거나 pop 하면, 다른 하나는 size가 그대로인 채 elements 배열만 똑같이 바뀌게 되는 것이다.

이 문제는 clone 안에서 clone을 호출하는, **clone을 재귀적으로 호출하는 방식**으로 해결할 수 있다. 위에서 본 스택의 경우에는 이렇게 재정의 할 수 있다.

```JAVA
@Override 
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone(); // elements 배열도 clone을 호출해주자.
        return result;
    } catch (ClassNotSupportedException e){
        throw new AssertionError();
    }
}
```

elements 배열에서 clone 을 호출해 그대로 적용해 줬다. 배열의 clone 메서드는 **런타임 타입과 컴파일타임 타입 모두 원본 배열과 똑같은 배열을 반환**하도록 이미 정의되어 있다. 사실 배열은 clone 기능을 제대로 사용하는 유일한 예이다.

하지만, 이는 elements 배열이 final이 아닌 경우에 사용 가능하다. 그런데 클래스를 이렇게 작성하게 되면 **'가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌**한다. Cloneable을 구현해 복사할 수 있는 클래스를 만드려면, 이 일반 용법을 어기고 final 한정자를 없애야 할 수도 있다.

2. 복잡한 가변 상태를 가지는 객체를 참조할 경우

만약 클래스가 다른 객체와 긴밀하게 연결되는 연결리스트의 노드처럼 복잡한 가변 상태를 가지는 경우, 그저 재귀적으로 clone을 호출하는 것은 부족할 수 있다.

다음은 연결리스트를 이용해 버킷 충돌을 해결하는 Seperate Chaining이 구현된 해시 테이블이다.

```JAVA
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
 
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
 
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
    ... // 생략
}
```

이 때, buckets 배열을 clone 메서드 안에서 `buckets.clone();` 으로 복사하게 될 경우, 복사된 buckets 배열의 Entry는 next 필드에 원본 buckets 배열의 next 노드를 참조하게 된다. 

이 문제를 해결하려면 next 필드도 재귀적으로 복사하도록 구현하면 된다.

Entry 클래스에 key, value, next를 똑같이 가지는 객체를 만들고, p.next가 null이 아닐 때까지 반복문을 돌면서, p.next도 똑같이 복사해 next에 담아주도록하는 deepCopy() 메서드를 만든다.

HashTable 클래스에서는 모든 버킷을 순회하며 연결리스트들을 deepCopy 해, 복사된 연결리스트들을 복사본의 버킷에 담는다.

```JAVA
public class HashTable implements Cloneable {

    private static class Entry {
        ... // 생략
        Entry deepCopy() {
            Entry result = new Entry (key, value, next); // 첫 노드를 새로 생성
            for (Entry p = result; p.next != null; p = p.next)
                p.next = new Entry(p.next.key, p.next.value, p.next.next); // 다음 노드를 새로 생성
            return result;
        }
    }
    @Override 
    public HashTable clone(){
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) // 원본의 복사본을 돌면서
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy(); // 연결리스트를 deepCopy
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    ... // 생략
}
```

혹은, 새로운 buckets 배열로 초기화 한 후 put(key, value) 메서드를 사용해 각각의 Entry를 담는 방법도 있다. 다만, 이 경우 put 메서드는 하위 클래스에서 재정의 되면 안되므로, final 키워드가 붙어야 한다. 만약 하위클래스에서 put이 재정의 됐다면, put 메서드를 HashTable의 clone 메서드에서 호출해도, 하위 클래스에서 재정의된 put이 호출되어, 원하는 상태가 아니게 될 수 있다.

### Cloneable의 대안

만약 상위 클래스에서 Cloneable 을 이미 구현했고, 이를 확장하는 경우라면 clone 을 구현해야한다. 하지만 이 경우가 아니라면 **복사 생성자와 복사 팩터리 메서드 방식**으로 더 나은 객체 복사 방식을 제공할 수 있다.

* 복사 생성자

    `public MyClass(Myclass originalInstance) {...};`

* 복사 팩터리 메서드

    `public static Myclass newInstance(Myclass originalInstance) {...};`

위 방법을을 이용해 생기는 장점은 다음과 같다:

* Java의 방식인 생성자를 이용하는 게 아닌, native 메서드에 의존하는 객체 생성을 더 이상 하지 않음.
* 강제력 없는 엉성한 규약에 기대지 않음.
* 생성자를 사용하기 때문에, 가변 객체를 final로 선언할수도 있음.
* 불필요한 CloneNotSupportedException을 던지지 않음
* super 를 호출하지 않아도 되기 때문에, 형변환 할 필요가 없음.
* 인자를 받을 수 있으므로, 팩터리 패턴의 장점을 살려 복제본의 내부 구현이 다르게 반환되도록 구현도 가능. 예컨데, 원래 HashSet 이었지만 인자에 따라 TreeSet으로 바꿔서 복제할 수도 있음.