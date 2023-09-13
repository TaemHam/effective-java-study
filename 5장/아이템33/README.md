# 아이템 33. 타입 안전 이종 컨테이너를 고려하라

   - 보통 제네릭을 사용하면 정해진 갯수의 타입 매개변수를 받을 수 있다.

   - 예를 들어 List\<T>는 타입 매개변수를 1개, Map\<K, V>는 타입 매개변수를 2개 받는다.

   - 만약 타입의 갯수를 동적으로 받고 싶은 자료구조를 선언하고 싶을 때는 타입 안전 이종 컨테이너를 고려해보자.

> 📌 **타입 안전 이종 (다른 종류) 컨테이너란?**<br>
> - 타입에 안전하면서 다른 종류의 타입을 받을 수 있는 컨테이너이다.
> - 이 때 컨테이너는 요소를 담을 수 있는 클래스라고 생각하면 된다.
> - 타입 안전 이종 컨테이너에는 Key, Value 구조로 이루어지는데 Key를 매개변수화 (제네릭으로 받음)하고 컨테이너에 값을 넣거나 뺄 때 매개변수화 한 키를 값과 함께 넣으면 된다.

<br>

> 📌 **타입 안전 이종 컨테이너란를 쓰는 목적!**<br>
> - 다양한 클래스의 타입을 Key로, 그에 따른 값을 1개의 클래스에서 관리하기 위함이다.

<br>

## 타입 안전 이종 컨테이너 예시

- 타입 안전 이종 컨테이너를 구현한 Favorites 클래스 작성하기
  - Map\<Class<?>, Object> 필드를 보자.
  
  - 이 필드는 class의 타입을 key로 받고, 모든 타입의 값을 받는다.
  
  - putFavorite는 Class 타입을 key로, 제네릭으로 제한한 타입의 값을 받아서 Map에 저장한다.
  
  - getFavorite는 Class 타입을 key로 저장된 값을 가져온다.

    <br>

    ```java
    static class Favorites {
        private Map<Class<?>, Object> favorites = new HashMap<>();

        public <T> void putFavorite(Class<T> type, T instance) {
            favorites.put(Objects.requireNonNull(type), instance);
        }

        public <T> T getFavorite(Class<T> type) {
            return type.cast(favorites.get(type));
        }
    }

- Favorites 클래스 사용하기
  
  - 1개의 Favorites 클래스를 선언 후 String, Integer, Favorites의 타입을 key로 각각의 값을 넣는다.
  
  - Class 타입을 key로 저장된 값을 가져온다.
 

    ```java
    public static void main(String[] args) {
        Favorites favorites = new Favorites();

        // 다양한 클래스 타입을 key로 넣어줄 수 있으므로 유연성이 높아진다.
        favorites.putFavorite(String.class, "Java");
        favorites.putFavorite(Integer.class, 12);
        favorites.putFavorite(Class.class, Favorites.class);

        String favoriteString = favorites.getFavorite(String.class);
        Integer favoriteInteger = favorites.getFavorite(Integer.class);
        Class<?> favoriteClass = favorites.getFavorite(Class.class);

        System.out.println("favoriteString = " + favoriteString);
        System.out.println("favoriteInteger = " + favoriteInteger);
        System.out.println("favoriteClass = " + favoriteClass);
    }
    ```
    > 📌 **참고**<br>
    > - favorites.getFavorite(String.class)의 String.class와 같이 컴파일 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 타입 토큰이라고 한다.
    > - 리터럴이란 값 그 자체를 의미한다.

    <br>

    > 📌 **주의**<br>
    > - 앞선 아이템에서 List<?>와 같이 비한정적 와일드카드 타입은 `값을 넣어주지는 못하지만 값을 꺼내오는 것은 가능`하다.
    > - Favorites의 favorites 필드는 Map의 key로 ? 가 아닌 Class<?>를 받는 것이므로 값을 입력할 수 있음을 헷갈리지 말자.
    > - 또 Favorites 클래스의 favorites 필드에서는 key에 입력된 class의 타입과 값의 타입이 무조건 동일하다고 보장되지 않는다.

    ```java
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        // 파라미터로 T 타입을 받지만 favorites에는 Object가 저장되므로 무조건 타입이 동일하다고 보장되지 않는다.
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> void putFavorite(Class<T> type, T instance) {
        // instance를 넣을 때 캐스팅해서 넣어주어도 논리적으로는 타입이 같다고 보장하지만, 컴파일러 입장에서는 반드시 동일하다고 볼 수는 없다.   
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    ```
<br>

## 타입 안전 이종 컨테이너의 제한사항 2가지

### 클라이언트에서 Key인 Class 객체를 로 타입으로 보내면?

- 클라이언트에서 Favorites를 다음과 같이 사용했다고 생각해보자.
    ```java
    favorites.putFavorite((Class) Integer.class, "문자열입니다.");
    ```
- 이러한 경우 Integer.class를 key로 문자열 값이 저장된다.

- 따라서 getFavorite 메서드 호출 시 ClassCastException이 발생하게 된다.

- 그러므로 아래 코드와 같이 put 메서드 호출 시 타입 캐스팅을 하면서 넣어주는 게 더 안정성을 높인다.

    ```java
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    ```

<br>

### 타입 안정 이종 컨테이너에는 실체화 불가 타입을 key로 사용할 수 없다.


```java
// List에 타입 파리미터를 넣은 클래스를 key로 사용하려하면 컴파일 오류가 발생한다!
favorites.putFavorite(List<String>.class, Collections.emptyList());

// List.class를 key로 넣으면 컴파일 오류는 발생하지 않지만 List<String>, List<Integer> 타입의 값도 모두 허용하므로 타입 안정성이 보장되지 않는다.
favorites.putFavorite(List.class, Collections.emptyList());
```

<br>

## 만약 Class<?> 가 아닌 한정적 타입 토큰을 사용하고 싶다면?

- Class<?>로 받는 타입 토큰은 어떠한 클래스도 타입 토큰으로 받을 수 있다.

- 만약 타입 토큰의 허용 범위를 제한하고 싶다면 한정적 타입 토큰을 사용하자.

```java
static class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();

    // putFavorite, getFavorite 메서드의 타입 파라미터를 Number의 하위 클래스로 제한하였다.
    public <T extends Number> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T extends Number> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

- 또는 asSubClass 메서드를 활용하여 타입을 제한하자.

  - Number 하위의 타입만 받고 싶을 때 아래와 같이 문자열로 className을 받고 asSubClass 메서드로 캐스팅과 동시에 ClassCastException이 발생하는 지 검증할 수 있다.
```java
public Number getFavorite(String className) {
    Class<?> classType;

    try {
        classType = Class.forName(className);
    } catch (ClassNotFoundException e) {
        throw new IllegalArgumentException(e);
    }

    return classType.asSubclass(Number.class).cast(favorites.get(classType));
}
```
<br>

# 추가학습
## Super Type Token

- 위의 token type 예제에서 List\<Integer> 등의 실체화 불가 타입은 Favorites에 key로써 넣을 수 없다고 했다. (타입 토큰으로 사용 불가능)

- super type token은 이를 해결하기 위한 방법이다.
  
  - 기본적으로 제네릭 타입은 런타임 시 소거되어 Object로 처리되는데 그걸 해결하여 제네릭 타입을 타입 토큰으로 사용하겠다는 의미이다.

- super type token은 타입 정보를 담은 제네렉 클래스를 1개를 익명 클래스로 선헌 후 key로써 사용하는 방식이다.

[super type token 알아보기](https://yangbongsoo.gitbook.io/study/super_type_token)

```java
public class SuperTypeToken {

    static class TypeSafeMap {
        Map<Type, Object> map = new HashMap<>();

        <T> void put (TypeReference<T> tr, T value) {
            map.put(tr.type, value);
        }

        <T> T get(TypeReference<T> tr) {
            if (tr.type instanceof  Class<?>)
                return ((Class<T>)tr.type).cast(map.get(tr.type));
            else
                return ((Class<T>)((ParameterizedType)tr.type).getRawType()).cast(map.get(tr.type));
        }
    }

    static class TypeReference<T> {
        Type type;

        public TypeReference() {
            Type sType = getClass().getGenericSuperclass();
            if (sType instanceof ParameterizedType) {
                this.type = ((ParameterizedType)sType).getActualTypeArguments()[0];
            } else {
                throw new RuntimeException();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        TypeSafeMap m = new TypeSafeMap();

        // TypeReference 익명 클래스를 선안하여 타입 토큰으로 사용한다.
        m.put(new TypeReference<String>(){}, "String");
        m.put(new TypeReference<Integer>(){}, 1);

        m.put(new TypeReference<List<Integer>>(){}, Arrays.asList(1,2,3));
        m.put(new TypeReference<List<String>>(){}, Arrays.asList("a", "b", "c"));

        System.out.println(m.get(new TypeReference<String>(){}));
        System.out.println(m.get(new TypeReference<Integer>(){}));
        System.out.println(m.get(new TypeReference<List<Integer>>(){}));
        System.out.println(m.get(new TypeReference<List<String>>(){}));
    }
}
```

<br>

### 스프링의 ParameterizedTypeReference
- 스프링에서는 제네릭을 타입 토큰으로 사용할 수 있도록 제공하는 ParameterizedTypeReference 클래스가 있다.

- ParameterizedTypeReference의 익명클래스를 사용하여 제네릭을 타입 토큰으로 사용한다.

```java
ResponseEntity<List<User>> exchange = restTemplate.exchange(
    "https://...",
    HttpMethod.GET,
    HttpEntity.EMPTY,
    // ParameterizedTypeReference의 익명 클래스를 선언하여 제네릭을 타입 토큰으로 사용할 수 있게 해준다.
    new ParameterizedTypeReference<List<User>>() {} 
);
```