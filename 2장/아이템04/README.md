# 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
## 문제점. 정적 필드와 정적 메소드만 담은 클래스를 만들고 싶은 경우

### 정적 필드와 정적 메소드만 담은 클래스는 왜 필요할까?
#### 1. 아이디어

- 반복되는 유틸성 필드나 메서드를 1개의 클래스에 작성하고 싶다.


#### 2. 유의점

- 정적 필드, 정적 메소드는 인스턴스를 선언하지 않고도 접근할 수 있으므로 인스턴스화가 되면 안 된다.

#### 3. 해결 방법

- 묵시적으로 생성되는 생성자가 아닌 private 접근 제한자를 사용하여 생성자를 명시한다.

    ```java
    public class UtilityClass {

        // 인스턴스화 방지를 위해 기본 생성자가 만들어지는 것을 막는다.
        private UtilityClass {
            throw new AssertionError(); 
        }

        // 정적 필드 예시
        public static String SERVER_IP = "localhost";

        // 정적 메소드 예시
        public static LocalDateTime getNow() {
            return LocalDateTime.now();
        }
    }
    ```

    > 📌 **참고!**
    > 
    > - 기본 생성자를 private로 선언하면 상속을 막는 효과도 있다.
    >   - 왜냐하면 상속을 받은 자식 클래스에서는 묵시적으로 부모의 기본생성자를 호출하기 때문이다.

<br>

### 인터페이스를 사용한 정적 필드만 작성하기
    
- 정적 필드를 선언한 인터페이스 만들기

    ```java
    public interface ServerInfo {

        String SERVER_IP = "localhost";
        int SERVER_PORT = 8080;
    }
    ```

    > 📌 **참고!**
    > 
    > - 인터페이스에 변수를 작성하면 기본값으로 public static final이므로 정적 변수로 활용할 수 있다.
    > - 인터페이스에 변수를 작성하면 상수 값이므로 다음과 같은 변수명 명명규칙을 따른다.
    >   - 변수명은 영어 대문자로 작성한다.
    >   - 의미를 구분 지을 때는 언더바로 구분한다.
<br>

<br>
<hr>

## 추가 학습
### 자바의 유틸성 라이브러리


- Apache Common

    [Apache Common 살펴보기](https://commons.apache.org/)


- Google Guava
    
    [Google Guava 살펴보기](https://github.com/google/guava)

<br>

### 스프링의 유틸성 라이브러리


- org.springframework.util 하위 클래스

    - StringUtils
    - FileCopyUtils
    - NumberUtils
    
    - ReflectionUtils
    - 등등..

- org.springframework.test.util 하위 클래스

    - ReflectionTestUtils
    - TestSocketUtils

    - AopTestUtils
    
    - 등등..

<br>

### Custom 유틸 클래스

- 내가 작성한 Custom 유틸 클래스

    ```java
    public class CommonUtils {

        private CommonUtils {
            throw new AssertionError();
        }

        // 람다를 이용하여 파라미터에 따른 where 조건 추가 시 NPE 방지 (QueryDsl에서 사용)
        public static BooleanBuilder nullSafeBuilder(Supplier<BooleanExpression> f) {
            try {
                return new BooleanBuilder(f.get());
            } catch (IllegalArgumentException e) {
                return new BooleanBuilder();
            }
        }
    }
    ```