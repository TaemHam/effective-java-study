# 아이템 12. toString을 항상 재정의하라

### 왜 toString을 항상 재정의해야 할까?
- Object의 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.
- 특히 String을 제외한 참조형 변수에 별도로 toString을 작성하지 않는다면 대부분이 객체의 해시코드를 16진수로 변환한 값을 반환한다.
- toString의 일반 규약에 따르면 `간결하면서 사람이 읽기 쉬운 형태의 유익한 정보`를 반환해야 한다.
- 또 toString의 규약은 `모든 하위 클래스에서 이 메서드를 재정의하라`라고 한다.

    > 📌 **toString를 반드시 재정의해야 하는지에 대한 의견**
    > - JavaDocs를 보면 아래와 같이 작성되어 있다.

    > - `[It is recommended that all subclasses override this method.]`

    > - 따라서 `모든 하위클래스에서 toString을 재정의하라`라는 의미가 아니라 `모든 하위클래스에서 toString을 재정의 것이 좋다` 라고 권장하는 뉘앙스로 해석된다.

    <br>

    [toString을 반드시 재정의해야 하는가?](https://stackoverflow.com/questions/13867269/when-should-i-override-tostring)

<br>

### toString은 어떻게 작성해야 할까?
- 작성해야 한다면 사람이 객체의 정보를 파악하기 용이하도록 작성해야 한다.
  
- toString은 그 객체가 가진 `주요 정보` 모두를 반환하는 것이 좋다.

<br>

### toString을 구현할 때는 반환값의 포맷을 문서화 할지 정해야 한다.

- 전화번호나 행렬 같은 값 클래스라면 문서화 하는 것이 좋다.

- 포맷을 명시하면 그 객체는 표준적이고 명확하고 사람이 읽을 수 있게 되기 때문이다.
  
- 띠라서 그 값 그대로 입출력에 사용하거나, csv 파일처럼 사람이 읽을 수 있는 데이터 객체로 저장할 수도 있다.

- 포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해주면 좋다.
 
  - 자바의 많은 값 클래스가 따르는 방식이며 예를들어 BinInteger나 BigDecimal 클래스가 있다.

    ```java
    public class BigInteger extends Number implements Comparable<BigInteger> {
        
        // 코드 중략..

        public String toString(int radix) {
            // 코드 중략..

            StringBuilder sb = new StringBuilder(numChars);

            // 정적 팩터리 메서드를 만들어서 포맷이 적용된 값을 출력한다!
            toString(abs, sb, radix, 0);

            return sb.toString();
        }
    }
    
    ```
- 반대로 포맷을 지정하게 되면 평생 그 포맷에 얽매이게 된다.
 
- 왜냐하면 한번 정해진 포맷대로 파싱하고, 새로운 객체를 만들고 데이터 객체를 만들기 때문이다.

- 따라서 반대로 포맷을 명시하지 않으면 향후 포맷을 개선할 수 있는 유연성을 얻게 된다.

<br>

### 포맷을 명시하든 아니든 의도를 명확하게 밝혀야 한다!

  - 포맷을 명시한 경우
    - 포맷을 명시할 땐 아주 정확하게 해야 한다.

    > 📌 **포맷 명시한 경우**
    > - 이 전화번호의 문자열 표현을 반환한다.

    > - 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.

    > - XXX는 지역 코드, YYY는 접두어, ZZZZ는 가입자 번호다.

    > - 각각의 대문자는 10진수 숫자 하나를 나타낸다.

    > - 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,

    > - 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면 전화번호의 마지막 네 문자는 "0123"이 된다.

    <br>
    
    ```java
    @Override
    public String toString() {
       return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
    ```

  - 포맷을 명시하지 않는 경우
  
    > 📌 **포맷 명시하지 않는 경우**
    > - 이 약물에 관한 대략적인 설명을 반환한다.

    > - 다음은 이 설명의 일반적인 형태이나,

    > - 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.

    > - "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"

    <br>
    
    ```java
    @Override
    public String toString() {
        // ...
    }
    ```

<br>

### 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.

- 즉 toString의 결과값에 포함된 정보들 (클래스의 필드 등)을 제공할 수 있는 getter 메소드를 제공하자는 의미이다.
  
- 그렇지 않으면 toString의 일부 값만 필요한 경우에 toString을 사용하여 다시 파싱해야 한다.
  
    > 📌 **만약 getter 메소드가 없다면..?**
    > - 클라이언트는 toString을 이용해 값을 파싱해야 한다.

    <br>

    ```java
    // getter 메소드를 제공하지 않는 경우
    // getPrefix가 아닌 toString을 파싱해야 하기 때문에
    // 불필요한 계산이 발생한다.
    String prefix = phoneNumber.toString().split("-")[0];
    ```

## 추가학습
### JPA의 엔티티에서 롬복의 @ToString을 쓸 때 주의점?

[JPA 엔티티에 연관된 엔티티가 있는 경우 주의점](https://stackoverflow.com/questions/23973347/jpa-java-lang-stackoverflowerror-on-adding-tostring-method-in-entity-classes)


