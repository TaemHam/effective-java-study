# 아이템 27. 비검사 경고를 제거하라

## 비검사 경고란 무엇인가?

- 비검사 경고란 타입 안정성을 보장할 수 없을 때 컴파일러가 발생시키는 경고이다.
  - 컴파일 에러가 아님에 주의!

[비검사 경고란 무엇인가?](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TechnicalDetails.html#FAQ001)

[비검사 경고를 사용하는 이유](https://stackoverflow.com/questions/30050574/advantage-of-suppresswarnings-annotation)

<br>

## 비검사 경고를 최대한 제거하자.

- 제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 된다.
  
- 컴파일러가 비검사 경고를 알려주고, 해결방법도 알려주므로 꼭 비검사 경고를 제거하도록 하자.

- 비검사 경고를 제거함으로써 타입 안정성을 보장하고, ClassCastException을 방지할 수 있다.

> **📌 참고**<br><br>
> - 비검사 경고를 `제거`하는 것은 잘못된 코드를 수정하여 비검사 경고가 나타나지 않도록 하는 것이다.
> - 비검사 경고를 `숨김`처리하는 것은 논리적으로 타입 안정성이 보장될 때 @SuppressWarnings를 사용해서 경고를 숨김처리하는 것을 의미한다.

```java
public class Main {
    // 컴파일러가 다음과 같은 경고를 발생시킨다.
    // Unchecked assignment: 'java.util.HashSet' to 'java.util.Set<java.lang.String>'
    public static void main(String[] args) {
        Set<String> set = new HashSet();
    }
}
``` 

```java
public class Main {
    // new HashSet() -> new HashSet<>()으로 변경
    // 즉 자바의 다이아몬드 연산자를 사용하여 비검사 경고를 제거하였다.
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();
    }
}
``` 
<br>

## @SuppressWarning으로 경고를 숨기기

- 비검사 경고를 제거할 수는 없는 상황인데 타입 안정성이 보장된 상황이라면 @SuppressWarning으로 비검사 경고를 숨기자.

- 그러나 타입 안정성이 보장되지 않았는데 비검사 경고가 보기 싫다고 숨기면 예상치 못한 에러가 발생할 수 있으므로 주의하자.

- @SuppressWarning를 사용할 때는 가능한 한 좁은 범위에 적용되도록 사용하자.

- 또 @SuppressWarning을 사용할 때 비검사 경고를 숨겨도 안전한 이유를 반드시 주석으로 남겨야 한다.

```java
public class Main {
    public static void main(String[] args) {
        Map<String, Object> map = new HashMap<>();
        // 비즈니스 로직 상 map에서 list를 조회해 올 수 있다고 한다면
        // 이 때 발생하는 비검사 경고를 @SuppressWarning으로 숨길 수 있다.
        List<String> list = (List<String>) map.get("key");
    }
}
```

- 다음은 ArrayList의 toArray 메소드이다.

- 교재에서는 메서드 안에 @SuppressWarning을 작성했지만 jav 17 버전에서는 메소드 바깥에 @SuppressWarning을 작성하였다.

<br>

```java
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // 타입 캐스팅 시 비검사 경고가 발생하여 @SuppressWarning으로 비검사 경고를 숨겼고,
        // 그 이유를 주석으로 작성해 두었다.
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```