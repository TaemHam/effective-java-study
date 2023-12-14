# 아이템 50. 적시에 방어적 복사본을 만들라

## 불변이란?

- 불변이란 객체가 생성 된 후 그 안의 내용물이 변경되지 않는 것이다.

- 객체가 불변이 아니라면 예상치 못하게 객체 내부의 값이 변경될 수 있으므로 반드시 방어적으로 프로그래밍 해야 한다.

* [불변이란?](https://www.geeksforgeeks.org/create-immutable-class-java/)

* [변경 가능성을 최소화하는 객체 만들기](https://github.com/TaemHam/effective-java-study/tree/887df1a99c2a1e3ff7cb7155505dd6ca7c60390e/4%EC%9E%A5/%EC%95%84%EC%9D%B4%ED%85%9C17)

<br>

## 불변식을 지키지 못하는 클래스

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }

        // start, end에 넘어온 날짜 데이터를 그대로 넣어주고 있음
        this.start = start;
        this.end = end;
    }

    //this.start & this.end 반환
    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }

    public String toString() {
        return start + " - " + end;
    }
}
```

- 위의 Period 클래스가 불변이 아닌 경우를 살펴보자.

```java
public static void main() {

    Date start = new Date();
    Date end = new Date();
    Period p = new Period(start, end);

    // 이 시점에 Period 내부의 end 값을 변경하였다.
    // Date 클래스가 불변이 아니기 때문에 발생한 문제이다.
    end.setYear(78);
}
```

- 따라서 날짜 관련 API를 사용할 때는 Instant, LocalDateTime, ZonedDateTime 같은 불변 클래스를 사용하도록 하자.

- 그러나 Date 클래스를 사용하는 많은 라이브러리들이 존재하므로 완전히 Date 클래스를 사용하지 않는 것은 불가능하다.

- 따라서 객체의 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사해야 한다.

<br>

## 생성자에서 방어적으로 복사하기

```java
// 매개변수의 방어적 복사본
public Period(Date start, Date end) {
    // 매개변수로 받은 Date 객체를 새로운 Date 객체로 만들어서
    // 외부에서 start나 end를 변경할 수 없도록 제한한다. (방어적 복사)
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
    }
}
```

> **📌 참고**<br>
>
> - 위의 예시에서 start, end를 방어적으로 복사 후 유효성 검사를 했다는 점에 주의하자.
> - 만약 멀티 쓰레드 환경에서 Period 객체를 사용한다고 가정하고, 유효성 검사를 하고 방어적으로 복사한다고 한다면 유효성 검사가 끝난 후, 새로운 변수가 할당되는 찰나에 다른 쓰레드가 원본 객체를 수정할 위험이 있기 때문이다.
> - 이를 TOCTOU 공격이라고 한다.
>   - [TOCTOU 공격이란?](https://m.blog.naver.com/gs_info/221575045619) >

<br>

- 방어적 복사를 할 때 clone을 쓰지 않은 것도 주의하자.

- Date 클래스는 final이 아니므로 clone이 Date가 정의한 게 아닐 수도 있다.
- 악의를 가진 사용자가 Date를 상속받은 클래스를 만들고, 그 자식 클래스에서 악의적인 clone 메서드를 작성하여 start와 end 필드에 접근할 수 있는 주소를 제공할 수도 있다.

- 따라서 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용하면 안된다.

> **📌 참고**<br>
>
> - Instant, LocalDateTime 은 final 클래스이므로 상속이 불가능하다.

<br>

### 접근자 (getter) 또한 방어적 복사본을 반환하자.

- 만약 악의적 공격자가 다음과 같이 공격하면 객체의 내부 값이 변경되어 버린다.

```java
// Period 인스턴스를 향한 두 번째 공격
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);
```

- 이러한 경우 다음과 같이 필드의 방어적 복사본을 반환하면 된다.

```java
// 수정한 접근자 - 필드의 방어적 복사본 반환
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

<br>

## 매개변수를 방어적으로 복사하는 또 다른 목적

- 매개변수를 방어적으로 복사하는 것은 불변 객체를 만들기 위해서만은 아니다.

- 생성자나 클라이언트가 제공한 객체의 참조 같은 매개변수로 전달된 객체를 내부의 자료구조에 보관해야 하는 경우를 생각해보자.

- 이러한 경우 내부의 자료구조에 저장한 객체가 변경되지 말아야하는 경우에는 복사본을 만들어서 저장해야 한다.

<br>

## 그렇다면 반드시 방어적 복사를 해야할까?

- 방어적 복사를 하는 것은 성능 저하가 따르고, 또 항상 쓸 수 있는 것도 아니다.

- 클라이언트에서 객체의 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있다.
