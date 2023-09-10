# 아이템 26. 로 타입은 사용하지 말라

## 로 타입이란?

> `제네릭` 클래스 에서 `타입 매개변수`를 전혀 사용하지 않은 것.

* 제네릭: 클래스를 정의할 때, 구체적인 타입(type)을 적지 않고 변수 형태로 적어 놓는 것. 

* 타입 매개변수: 제네릭 클래스에서 타입을 변수처럼 대입해 사용하는 매개변수.

    * 종류: <br>
        E – Element <br>
        K – Key <br>
        N – Number <br>
        T – Type <br>
        V – Value <br>

```JAVA
// 자바 List 인터페이스
public interface List<E> extends Collection<E> {
    ...
}
```

쉽게 말해, 로 타입은 다이아몬드라고 불리는 `<>`, 이게 붙지 않고 만들어진 제네릭 클래스 인스턴스를 의미한다. 

```Java

public static void main(String[] args) {
    List rawList = new ArrayList();
}

```

### 왜 로 타입을 사용하면 안될까?

* 디버깅이 힘들어진다. 
    
    타입 안정성이 떨어지기 때문이다. 해당 인스턴스에 의도하지 않은 타입이 실수로 들어가도, 컴파일 시에 에러를 내지 않고, 런타임에, 그것도 대부분은 실수한 코드와 먼 곳에서 에러가 발생한다. 디버깅을 어렵게 하는 원인이 되어버린다.

    제네릭 타입을 사용하면 컴파일러가 해당 인스턴스가 사용할 타입을 알 수 있고, 컴파일 할 때 타입 오류를 잡아낼 수 있게 되는 것이다.

    ```JAVA
    class FailingClass {
        public static void main(String[] args) {
            List<BigDecimal> bigNumbers = new ArrayList<>();

            ...

            BigInteger bigNumber = new BigInteger(98765432123456789) // 데시멀이 아닌 인티저다.

            doSomethingWithList(bigNumbers, bigNumber);

            ...

            BigDecimal s = bigNumbers.get(0); // 런타임 오류가 여기서 나게 된다.
        }

        private static void doSomethingWithList(List list, Object o) {

            ...

            list.add(o);
        }
    }
    ```

### 그렇다면 왜 로 타입이 존재하는 걸까?

바로 호환성 때문이다.

자바는 첫 출시 이후 제네릭이 나오기까지 10년이 걸렸는데, 그 기간 동안 이미 제네릭 없이 짠 코드가 너무 많아져 버렸다. 로 타입은 제네릭을 도입하면서도 기존 코드가 문제없이 동작하도록 하기 위해 생겨난 것이다.

### 이런 경우엔 로 타입 써도 되지 않을까?

1. 모든 객체를 허용하는 게 의도한 거라서, \<Object>를 붙이지 않고 로 타입을 쓰고 싶은 경우

안된다. 제네릭으로 \<Object>를 넣어 선언하는 것은 컴파일러에게 Object를 허용한다고 의도를 전달한 것이지만, 로 타입으로 선언하면 타입 체크를 완전히 포기한 것이다.

예를 들어, List로 선언된 변수는 List\<String> 객체도, List\<Integer> 객체도 받을 수 있지만, List\<Object>로 선언된 변수는 List\<Object>를 제외한 다른 객체는 받을 수 없다.

2. 어차피 원소는 건드리지도 않을 거라서, 원소의 타입을 몰라도 되는 경우 

역시 안된다. 아무리 인터페이스 작성자가 원소를 건드리지 말라고 해도, 메서드 매개변수를 로 타입으로 작성한다면, 다른 타입의 원소를 넣는걸 막을 수 없다. 이를 '타입 불변식이 훼손된다'고 한다.

```JAVA
public interface RawTypeInterface {
    void doSomethingWithSetSize(Set set);
}

class RawTypeInterfaceImpl implements RawTypeInterface {

    @Override
    public void doSomethingWithSetSize(Set set) {
        int setSize = set.size();

        ...

        set.add(1); // 원래 Set이 <String>을 담고 있어도, 오류를 내지 않는다.
    }
}

```

제네릭으로 만들면 이를 막을 수 있는 방법이 존재한다. 바로 `비한정적 와일드카드 타입`을 사용하는 것이다. 비한정적 와일드카드 타입을 사용해 만들어진 컬렉션에는 null 외에는 어떤 원소도 넣을 수 없다. 비한정적 와일드카드는 `?`로 표시한다.

```JAVA
public interface UnboundWildcardTypeInterface {
    void doSomethingWithSetSize(Set<?> set);
}

class UnboundWildcardTypeInterfaceImpl implements UnboundWildcardTypeInterface {

    @Override
    public void doSomethingWithSetSize(Set<?> set) {
        int setSize = set.size();

        ...

        set.add(1); // 여기서 컴파일 오류를 낸다.
    }
}
```

### 그럼 예외 없이 로 타입은 쓰면 안되는 걸까?

딱 두 가지 예외가 존재한다.

1. 객체의 클래스 자체를 확인하는 `class 리터럴`

    `List.class`, `Set.class` 등

2. 계층구조를 확인하는 `instanceof 연산자`  

    `if (o instanceof Set)` 등

> **📌 참고**<br><br>
> 다음은 5장 내내 사용할 용어에 관한 정리 표이다.
> |한글 용어|영문 용어|예시|등장 장소|
> |:-:|:-:|-|-|
> |매개변수화 타입|parameterized type|List\<String>|아이템 26|
> |실제 타입 매개변수|actual type parameter|String|아이템 26|
> |제네릭 타입|generic type|List\<E>|아이템 26, 29|
> |정규 타입 매개변수|formal type parameter|E|아이템 26|
> |비한정적 와일드카드 타입|unbounded wildcard type|List\<?>|아이템 26|
> |로 타입|unbounded wildcard type|List|아이템 26|
> |한정적 타입 매개변수|bounded type parameter|\<E extends Number>|아이템 29|
> |재귀적 타입 한정|recursive type bound|\<T extends Comparable\<T>>|아이템 30|
> |한정적 와일드카드 타입|bounded wildcard type|List\<? extends Number>|아이템 31|
> |제네릭 메서드|generic method|static \<E> List\<E> asList(E[] a)|아이템 30|
> |타입 토큰|type token|String.class|아이템 33|