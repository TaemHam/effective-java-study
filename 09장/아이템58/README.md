# 아이템 58. 전통적인 for 문보다는 for-each 문을 사용하라

컬렉션을 반복문으로 탐색할 때, 스트림을 사용하면 프로그램이 짧고 깔끔해지는 장점이 있지만<br>
무조건 스트림을 사용한다고 좋아지는 건 아니다. ([아이템 45](https://github.com/TaemHam/effective-java-study/tree/main/7%EC%9E%A5/%EC%95%84%EC%9D%B4%ED%85%9C45))

그럼 for 문은 어떤 식으로 사용하는 게 좋을까?

다음 두 가지 for 문을 보자.

```JAVA
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ...
}
```

```JAVA
for (int i = 0; i < a.length; i++) {
    Element e = a[i];
    ...
}
```

우리가 필요한 건 컬렉션 안의 원소일 뿐인데, 원소를 얻기 위해 Iterator 나 int 를 받는 새로운 변수를 만든다.<br>
따라서 이 방법들은 반복자와 인덱스 변수를 코드를 지저분하게하고 오류가 생길 가능성이 높아진다.

### for-each 문

위의 단점은 for-each 문을 사용함으로 해결할 수 있다.

```JAVA
for (Element e:  elements) {
    ...
}
```

새로운 변수 int 나 Iterator를 다룰 일이 없어 헷갈릴 일도 없고, 또 elements 가 배열인지 컬렉션인지 신경 쓸 필요도 없이 Element 만 가져다 사용하면 된다.

또, 만약 컬렉션을 중첩해 순회해야 한다면 for-each문의 이점은 더욱 커진다.

다음 버그가 있는 코드를 보자.

```JAVA
    enum Suit { CLUB, DIAMOND, HEART, SPADE }
    enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
        NINE, TEN, JACK, QUEEN, KING }

    ...

    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());

    Card(Suit suit, Rank rank ) {
        this.suit = suit;
        this.rank = rank;
    }

    public static void main(String[] args) {
        List<Card> deck = new ArrayList<>();
        
        for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
                deck.add(new Card(i.next(), j.next()));
    }
```

이 코드의 문제는 `i.next()` 가 중첩된 for 문 안에서 호출되고 있다는 데에 있다.

만약 i의 Suit 가 바닥이 나면, 런타임 에러로 `NoSuchElementException`를 뿜어내게 될 것이다.

만약 향상된 for 문을 사용한다면, 이런 걱정은 할 필요도 없게 된다.

```JAVA
    public static void main(String[] args) {
        List<Card> deck = new ArrayList<>();
        
        for (Suit suit: suits)
            for (Rank rank: ranks)
                deck.add(new Card(suit, rank));
    }
```

### for-each 문을 사용할 수 없는 상황

하지만 안타깝게도 for-each 문을 사용할 수 없는 상황이 세 가지 존재한다.

- 파괴적인 필터링(destructive filtering)<br>
  컬렉션을 순회하면서 컬렉션 내의 원소를 제거해야 한다면, Iterator의 remove 메서드를 호출해야 한다. 그렇지 않으면 `ConcurrentModificationException`을 마주하게 된다.<br>
  자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

- 변형 (transforming)<br>
  리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

- 병렬 반복(parallel iteration)<br>
  여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 명시적으로 제어해야 한다.

### for-each 문을 사용하기 위한 조건

직접 만든 컬렉션이나 객체로 for-each 문을 사용하려면 `Iterable` 인터페이스를 구현해두어야 한다.

```Java
public interface Iterable<E> {
    Iterator<E> iterator();
}
```

`Iterable` 인터페이스는 iterator 메서드 하나 뿐이라 간단해 보이지만, 사실 반환 타입으로 사용할 `Iterable`도 구현해야 하니 조금 까다로운 점이 있긴 하다.

하지만 구현해두면 for-each 문의 장점을 가져갈 수 있으니, 구현해보는 걸 고려해 보는 것도 좋다.