# 아이템 56. 공개된 API 요소에는 항상 문서화 주석을 사용하라

API를 쓸모 있게 하려면 그 API 사용법에 대해 잘 작성된 문서도 있어야 한다. <br>
문서가 잘 갖춰지지 않은 API는 쓰기 헷갈려서 오류의 원인이 되기 쉽다.<br>

API 문서를 직접 작성하게 되면, 문서 포맷을 정하는 것부터 API 수정에 따라 바뀐 문서도 다시 만들어야 한다.

하지만 자바에서는 이 귀찮은 작업을 도와줄 자바독 유틸리티가 있다.

## 자바독 (Javadoc)

자바독은 API 문서화 유틸리티로, 소스코드 파일에서 문서화 주석(자바독 주석)이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.

<details>
<summary>String 클래스에 붙은 자바독의 생김새</summary>

![자바독의 생김새](https://github.com/TaemHam/effective-java-study/assets/95671168/9cb6b31f-75e6-4892-830c-56f24e93a22e)
</details>

API를 올바르게 문서화 하려면, 공개된 모든 **클래스, 인터페이스, 메서드, 필드 선언** 모두에 문서화 주석을 달아야 한다.

### 자바독 생성법

자바독을 만드는 방법은 정말 간단하다. 

터미널에서 다음 명령어를 입력하면 끝이다.

```bash
$ javadoc -d docs {file_name}
# 만약 한글이 포함된다면
$ javadoc -d docs {file_name} -encoding UTF-8 -charset UTF-8 -docencoding UTF-8
```

그러면 [이런](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html) 생김새의 페이지가 나오는 html 파일을 자동으로 생성해준다. 잘 보면 위의 이미지와 같은 내용이 포함되어 있다.

### 자바독 주석 다는 법

* 앞에 `/**` 으로 시작하는 주석을 달고, 모든 줄 처음에 `*`를 붙이고, 마지막은 `*/` 을 붙여 끝낸다.<br>
IntelliJ를 사용한다면, `/**` 를 치고 엔터를 치는 순간, 기본적인 틀을 갖춘 자바독 주석이 생성된다.

* 결국은 html 파일로 나오는 것이니, 기본적으로 모든 html 태그를 사용할 수 있다. <br>
  그 외에도 다음과 같은 주석을 포함시킬 수 있다.
    
    * `{@summary ...}`

        * 요약 설명을 위한 태그다.
        * 주로 첫 문장에 쓴다.

    * `{@code ...코드... }`

        * 태그로 감싼 내용을 코드용 폰트로 렌더링한다.
        * 태그로 감싼 내용에 포함된 HTML요소나 다른 자바독 태그를 무시한다.
        * `<pre>{@code ...코드... }</pre>` 와 같이 사용하면 마크다운의 코드블럭처럼 여러줄로 된 코드도 작성 가능하다. <br>
        단, 그 안에 `@`을 쓸땐 탈출문자를 붙여야 한다.

    * `{@literal ... }`

        * <, >, &등의 HTML 메타문자를 포함시킨다.
        * `{@code ...코드... }` 와 비슷하지만 코드 폰트로 렌더링하진 않는다.

    * `{@index ... }`

        * 중요한 용어를 추가로 색인화할 수 있다.

* 클래스에는 다음의 주석을 붙일 수 있다.

    ```JAVA
    /**
     * ...
     * @author  Josh Bloch
     * @author  Neal Gafter
     * @see Collection
     * @see Set
     * @see ArrayList
     * @see LinkedList
     * @see Vector
     * @see Arrays#asList(Object[])
     * @see Collections#nCopies(int, Object)
     * @see Collections#EMPTY_LIST
     * @see AbstractList
     * @see AbstractSequentialList
     * @since 1.2
     */
    public interface List<E> extends Collection<E> { ...
    ```

    * `@author` : 코드 소스 작성자를 명시한다.
    * `@version` : 구현체, 패키지 버전을 명시한다.
    * `@deprecated` : 해당 클레스(구현체)의 삭제 또는 지원이 중단되었음을 알려준다.
    * `@since` : 해당 클래스가 추가된 버전을 명시한다.

* 메서드에는 다음 주석을 붙일 수 있다.

    ``` JAVA
    /**
     * Returns the element at the specified position in this list.
     *
     * <p>This method is <i>not</i> guaranteed to run in constant
     * time. In some implementations it may run in time proportional
     * to the element position.
     *
     * @param  index index of element to return; must be
     *         non-negative and less than the size of this list
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= this.size()})
     */
    E get(int index) { ...
    ```

    * `@param`

        * 전제조건에 영향받는 매개변수에 붙인다.
        * 모든 매개변수에 붙이는게 좋다.
        * 관례상 명사구를 쓰고, 마침표를 붙이지 않는다.

    * `@return`

        * 반환타입이 void가 아니라면 붙인다.
        * void가 아니라도 메서드의 설명과 일치할 경우엔 생략해도 된다.
        * 관례상 명사구를 쓰고, 마침표를 붙이지 않는다.

    * `@throws`

        * 비검사 예외를 선언할 때 붙인다.
        * 비검사 예외 하나당 전제조건 하나를 연결한다.
        * 관례상 마침표를 붙이지 않는다.

    * `@implSpec`
        
        * 상속용으로 설계된 클래스일 때, 해당 메서드와 하위 클래스 사이의 계약을 설명한다.
        * 하위 클래스들이 그 메서드를 상속하거나 super키워드를 이용해 호출할 때 그 메서드가 어떨게 동작하는지를 명확히 인지하고 사용하도록 해야한다.

* 제네릭이라면 모든 타입 매개변수에 주석을 달아야 한다.

    ```JAVA
    /**
     * An object that maps keys to values.  A map cannot contain duplicate keys; 
     * ...
     * @param <K> the type of keys maintained by this map
     * @param <V> the type of mapped values
     */
    public interface Map<K,V> { ...
    ```

* 열거 타입이나 상수들에는 각각 주석을 달아야 한다.

    ```JAVA
    /**
     * An instrument section of a symphony orchestra.
     */
    public enum OrchestraSection {
        /** Woodwinds, such as flute, clarinet, and oboe. */
        WOODWIND,

        /** Brass instruments, such as french horn and trumpet. */
        BRASS,

        /** Percussion instruments, such as timpani and cymbals. */
        PERCUSSION,

        /** Stringed instruments, such as violin and cello. */
        STRING;
    }
    ```

* 애너테이션 타입에는 타입 자체와, 멤버들에도 모두 주석을 달아야 한다.

    ```JAVA
    /**
     * Indicates that the annotated method is a test method that
     * must throw the designated exception to pass.
     */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
        /**
         * The exception that the annotated test method must throw
         * in order to pass. (The test is permitted to throw any
         * subtype of the type described by this class object.)
         */
        Class<? extends Throwable> value();
    }
    ```

### 자바독 주석 작성시 유의사항

* how가 아닌 what을 기술해야 한다. (어떻게 동작하는지가 아니라 무엇을 하는지 기술해야 한다.)

* 클라이언트가 해당 메서드를 호출하기 위한 전제조건과 사후조건을 모두 나열해야 한다.

* 전제조건은 `@throws`로 비검사 예외를 선언한다. 비검사 예외 하나당 전제조건 하나와 연결된다.

* `@param`으로 그 전제조건에 영향받는 매개변수에 기술한다.

* 첫 문장은 주로 요약 설명이다.

    * 주어가 없는 동사구여야 한다. 2인칭 문장(Return)이 아닌 3인칭 문장(Returns)을 사용해야 한다.
    * 한 클래스 혹은 한 인터페이스 안에 요약설명이 중복되면 안된다. (특히 오버로딩된 메서드들에서 특히 조심하자)

* 마침표에 주의해야 한다.

    * 만약 문장 중간에 꼭 삽입되어야 한다면, `{@literal ... }`을 이용하자.

        ```JAVA
        /**
         * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
        */
        ```

* 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.

* 직렬화 할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.