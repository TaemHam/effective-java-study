# 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

## 일련의 원소를 반환하는 방법

- 일련의 원소 (원소 시퀀스)를 반환하는 방법은 여러가지가 있다.

  > 📌 **참고**
  >
  > - 일련의 원소 (원소 시퀀스)는 쉽게 말해 데이터를 여러 개 담을 수 있는 타입이라고 생각하자.
  > - 예를 들어 컬렉션, 배열, 스트림 등이 있다.

- 자바 7까지는 다음과 같은 원소 시퀀스를 반환할 수 있었다.
  - Collection, Set, List 같은 컬렉션 인터페이스
  - Iterable
  - 배열

<br>

## 스트림은 반복을 지원하지 않는다.

- 사실 스트림이 반복을 지원하지 않는다는 것은 45장에서 학습하였다.

- 따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.

  > 📌 **참고**
  >
  > - 스트림은 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고 Iterable 인터페이스가 정의한 방식대로 동작한다. (구현이 아니다.)
  > - 그러나 forEach로 스트림을 반복하지 못하는 이유는 스트림이 Iterable을 extend 하지 않았기 때문이다.
  >   - 참고로 자바에서 반복문을 수행하려면 원소 시퀀스는 iterable이어야 한다.

<br>

## 스트림을 반복하는 방법 2가지

- 스트림을 반복하려면 스트림을 Iterable로 형변환하자.

```java
public static void main(String[] args) {
    Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);

    // 스트림을 iterator로 형변환하여 for문에서 사용한다.
    for (Integer num : (Iterable<Integer>) stream::iterator) {
        System.out.println("num = " + num);
    }
}
```

- 스트림과 Iterable을 중개해주는 어댑터 메서드 사용하기.

```java
// 스트림을 iterable로 변환해주는 어댑터 메서드
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

public static void main(String[] args) {
    Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);

    for (Integer num : iterableOf(stream)) {
        System.out.println("num = " + num);
    }
}
```

<br>

## API를 만들 때, Stream을 반환? Iterable을 반환?

### Stream을 반환하는 경우

- 이 메서드가 오직 스트림 파이프 라인에서만 쓰일 걸 안다면 스트림을 반환하자.

- Stream을 iterable로 변환하는 어댑터 메서드

  - 클라이언트는 반환받은 스트림을 iterable로 변환하여 사용할 수 있다.

  ```java
  // 스트림을 iterable로 변환해주는 어댑터 메서드
  public static <E> Iterable<E> iterableOf(Stream<E> stream) {
      return stream::iterator;
  }
  ```

### Iterable을 반환하는 경우

- 이 메서드가 오직 반복문에서만 쓰일 걸 안다면 Iterable을 반환하자.

- Stream을 iterable로 변환하는 어댑터 메서드

  - 클라이언트는 반환받은 iterable을 스트림으로 변환하여 사용할 수 있다.

  ```java
  // iterable을 스트림으로 변환해주는 어댑터 메서드
  public static <E> Stream<E> streamOf(Iterable<E> iterable) {
      return StreamSupport.stream(iterable.spliterator(), false);
  }
  ```

<br>

## Collection을 반환하자

- Collection은 Iterable의 하위 타입이며, stream 메서드도 제공한다.

- Collection은 Iterable을 지원하면서도 스트림도 동시에 지원하는 것이다.

- 따라서 일련의 원소(원소 시퀀스)를 반환하는 공개 API에서는 Collection이나 그 하위 타입을 반환하는 것이 좋다.

  - 왜냐하면 클라이언트가 iterable이나 stream을 쉽게 선택하여 반환값을 사용할 수 있기 때문이다.

  > 📌 **주의**
  >
  > - 그러나 Collection의 크기가 엄청 커지는 경우 메모리에 올리면 안되므로 반환 타입을 어떻게 할지 반드시 고려해야 한다.

<br>

## Collection이 너무 커서 전용 컬렉션을 구현해야 하는 예제

- 다음은 주어진 집합의 멱집합 (한 집합의 모든 부분 집합을 원소로 하는 집합)을 반환하는 경우이다.

- 예를 들어 원소의 갯수가 n개면 멱집합의 원소의 갯수는 2^n개이다.

- 이러한 경우 Java에서 제공하는 List, Set 등의 기본 컬렉션을 사용하는 것이 아닌 별도의 컬렉션을 직접 정의하는 것이 낫다.

```java
public class PowerSet {
	public static final <E> Collection<Set<E>> of(Set<E> s) {
        // 원소를 담은 리스트
		List<E> src = new ArrayList<>(s);

        // Collection의 size는 int 타입이므로 2^31 - 1 크기까지의 size를 가질 수 있다.
		if (src.size() > 30) {
			throw new IllegalArgumentException("집합에 원소가 너무 많습니다.(최대 30개)".: + s);
        }

		return new AbstractList<Set<E>>() {
			@Override public int size() {
				// 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱과 같다.
				return 1 << src.size();
			}

			@Override public boolean contains(Object o) {
				return o instanceof Set && src.containsAll((Set)o);
			}

			@Override public Set<E> get(int index) {
				Set<E> result = new HashSet<>();
				for (int i = 0; index != 0; i++, index >>=1)
					if ((index & 1) == 1) {
						result.add(src.get(i));
                    }
					return result;
			}
		};
	}
}
```

<br>

## 결론

- 일련의 원소 (원소 시퀀스)를 반환하는 경우에는 클라이언트가 스트림, iterable 중 편한 것을 쉽게 선택할 수 있게 Collection으로 반환하는 것이 좋다.

- 만약 반환하는 컬렉션에 담기는 원소가 적으면 ArrayList 같은 기본 컬렉션에 담아서 반환하자.

- 만약 앞선 예제인 멱집합처럼 컬렉션에 담기는 원소가 많아 메모리 이슈가 생길 가능성이 있으면 전용 컬렉션을 별도로 구현할지를 고민하자.

- 컬렉션을 반환하는 것이 불가능하면, 스트림과 iterable 중 반환했을 때 더욱 자연스러운 것을 반환하자.
