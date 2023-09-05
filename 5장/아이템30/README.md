# 아이템 30. 이왕이면 제네릭 메서드로 만들라

  > 📌 **참고**
  > 한 번 다시 짚고 넘어가야 할 점: 그래서 왜 제네릭을 쓸까?
  > 똑같은 코드를 적용하고 싶은 타입마다 중복해서 작성하지 않고, 중복된 건 빼기 위해서.

## 제네릭 메서드

Collection 의 binarySearch 나 sort 등의 정적 유틸리티 메서드들은 보통 제네릭으로 작성한다.

다음은 제네릭이 아닌, 로 타입으로 작성한 집합의 합집합 연산 메서드이다.

```Java
 public static Set union(Set s1, Set s2){
        Set result = new HashSet(s1);
        result.addAll(s2);
        return result;
    }
```

컴파일엔 문제가 없겠지만, s1 과 s2의 원소의 타입이 다른 경우가 생길 수 있어 타입 안전하지 않다. 따라서 자바는 unchecked 타입 경고를 발생시킨다.

경고를 없애는 건 쉽다. 메서드에서 사용하는 모든 Set에 <E>를 붙여 타입 매개변수로 명시하면 된다.

```Java
public static <E> Set<E> union(Set<E> s1, Set<E> s2){
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }

```

## 제네릭 싱글턴 팩터리

싱글턴으로 만든 불변 객체 인스턴스를 여러 타입으로 활용하고 싶을 때가 있다. 제네릭은 런타임에 타입 정보가 소거(Item 28)되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.

조건이 있다면, 요청한 타입 매개변수에 맞게 싱글턴 객체의 타입을 바꿔주는 정적 팩터리 메서드를 만들어야 한다.

그 정적 팩터리 메서드를 **제네릭 싱글턴 팩터리**라고 부른다. Collections.reverseOrder 와 Collections.emptySet이 이 방식을 사용한다.

```JAVA
public static <T> Comparator<T> reverseOrder() {
// T 타입으로 캐스팅된 Singleton ReverseComparator 반환, 타입정보는 소거됨으로 그때그때 캐스팅만 되는 것
        return (Comparator<T>) ReverseComparator.REVERSE_ORDER; 
}

public static final <T> Set<T> emptySet() { 
  		return (Set<T>) EMPTY_SET; 
}
```

이번에는 항등함수 (identity function)를 담은 클래스를 고려해보자.

항등함수 객체는 상태가 없으므로 요청할 때마다 새로 생성하는 것은 낭비다.
(상태가 없을 수밖에 없다. 입력으로 들어온 것을 그대로 다시 내보내줘야 하니.)

자바의 제네릭이 실체화된다면 항등함수를 타입별로 하나씩 만들어야 했겠지만,
소거 방식을 사용한 덕에 제네릭 싱글턴 하나면 충분하다.

```Java
public class GenericSingletonFactory {
    private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }
}
```

* @SuppressWarning 을 사용한 이유

  위 코드에서 T가 어떤 타입이든 UnaryOperator가 UnaryOperator가 아니기 때문에 경고가 발생한다.

  하지만 항등함수란 입력 값을 수정 없이 그대로 반환하는 특별한 함수, 그러므로 T가 어떤 타입으로 들어오던 UnaryOperator를 사용해도 타입 안전하다는 것을 개발자는 안다.

  그렇기에 SuppressWarinings 어노테이션을 추가해 경고를 지워줌으로써 깔끔하게 컴파일 할 수 있다. 

  > 📌 **참고**
  > 들어온 매개변수 그대로 내보내주는 항등 함수는, 보통 `Function.identity()` 으로 쓰고,
  > 객체 리스트를, 특정 필드를 키로 갖는 맵으로 변환하는 `Collectors.toMap()` 등의 작업을 할 때 사용한다.
  > ```Java
  > Map<Integer, <Member> memberMap = members.toArray().stream()
  >     .collect(Collectors.toMap(Member::getId, Function.identity())
  > );
  > ```

## 재귀적 타입 한정

드문 경우긴 하지만, 자기 자신이 들어간 표현식을 사용해 타입 매개변수의 허용 범위를 한정할 수 있다.

이를 재귀적 타입 한정 (Recursive type bound) 이라고 부른다. (`<E extends MyClass<E>>`)

재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰인다.

```JAVA
public interface Comparable <T>	{
  	int compare(T o);
}
```

위에서 타입 매개변수 T는 Comparable를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.

쉽게 말해 String 은 String 끼리만 비교할 수 있도록 해놨다는 말이다.

이제 재귀적 타입 한정을 사용해 컬렉션 원소 중 최댓값을 구하는 `max()` 메서드를 작성한다면, 다음과 같이 작성할 수 있다.

```Java
public class RecursiveTypeBound {
    public static <E extends Comparable<E>> E max(Collection<E> c) {
        if (c.isEmpty())
            throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

        E result = null;
        for (E e : c)
            if (result == null || e.compareTo(result) > 0)
                result = Objects.requireNonNull(e);

        return result;
    }
    public static void main(String[] args) {
        List<String> argList = Arrays.asList(args);
        System.out.println(max(argList));
    }
}
```

위 코드는 컬렉션에 담긴 원소에 순서를 기준으로 최댓값을 계산한다.

컴파일 오류나 경고는 발생하지 않는다.

재귀적 타입 한정은 프로그램을 복잡하게 할 가능성도 있지만, 그런 일은 잘 일어나지 않는다.