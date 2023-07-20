# 아이템 4. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

사용하는 자원에 따라 동작이 달라지는 클래스는 **정적 유틸리티나, 싱글턴 방식이 적합하지 않다**.<br>
다음은 정적 유틸리티와 싱글턴으로 구현한 문법 검사기 코드이다.

### 정적 유틸리티
```JAVA
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {
    }
    ...
}
```
### 싱글턴
```JAVA
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {
    }

    public static SpellChecker INSTANCE = new SpellChecker();
    ...
}
```
</details>

### 왜 적합하지 않을까?

1. 유연하지 않기 때문

사용하는 자원의 종류가 여러가지가 될 수 있다면, 자원의 종류마다 새로운 클래스를 만들게 된다. 위의 문법 검사기를 예로 들면, 언어의 종류마다 새로운 dictionary를 담는 SpellChecker를 만들게 되는 것이다.

2. 테스트가 어렵기 때문

단위 테스트를 작성하려 해도, 자원이 클래스에 묶여있다면 Mock 객체로 대체할 수 없어 테스트가 어려워 진다. 위의 Lexicon이 정상적으로 동작하는 경우와 오류를 내는 경우 SpellChecker가 어떻게 동작하는지 테스트 하고 싶어 Mock 객체를 넣어주고 싶어도, 위의 두 방식으로는 그렇게 할 수 없다. 

### 그럼 어떻게 수정해야 좋을까?

인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이 적합하다.

```JAVA
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary){
    	this.dictionary = Objects.requireNotNull(dictionary);
    }
    
    ...
}
```

위와 같이 자원으로 사용할 객체를 생성자에 넣어 초기화 해주는 방식을 사용하면,

1. 유연성 해결<br>
    Lexicon을 구현한 다른 언어의 사전을 사용하는 것으로 새로운 문법 검사기를 만들 수 있다.

2. 테스트 해결<br>
    Lexicon의 Mock 객체를 넣어서 딱 SpellChecker의 코드만 테스트할 수 있다. 

또 한 단계 더 들어가서, 의존객체를 통째로 넘겨주는 것이 아니라, 의존객체를 생성하는 Factory를 넘겨주게 할 수도 있다. 자바8 에서는 특히 이것을 명확하게 나타낼 수 있는데, Supplier 를 아래처럼 이용하는 것이다.

```JAVA
    public SpellChecker(Supplier<? extends Lexicon> dictionaryFactory) {
        this.dictionary = dictionaryFactory.get();
    }
```