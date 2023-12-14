# 아이템 60. 정확한 답이 필요하다면 float 와 double은 피하라

## 부동소수점

컴퓨터는 소수점을 가지는 실수들을 2진수로 표현한다.

![실수 표현 방식](https://steemitimages.com/640x0/https://steemitimages.com/DQmeKeVdmnTn5KAHD6aVk1x3ZkfWzM6XzW37efVd3rEtHxD/binary.png)

하지만 0.3 같이 이진수로 표현하면 0.01001100110011... 의 무한소수가 나오는, 0.5, 0.25 등을 더하는 걸로 계산하기 어려운 실수들도 있다.

그럼 자바는 이걸 어떻게 저장할까?

### float & double

먼저, 과학과 공학 계산용으로 만들어져, 빠르지만 소수의 근사치만을 저장하는 `float`와 `double` 타입이 있다.

이 두 타입들은 부동소수점 방식으로 실수를 저장한다.

![부동소수점](https://steemitimages.com/DQme3vRe1nGigGs1GfZkU5ffbufAs1gSNT4MKqR7F1PcxCi/IEEE754.png)

이 부동소수점 방식으로 저장하게 되면 23비트 이후의 부분들은 지워져서, 유한한 수를 다루기 때문에 계산은 빨라지지만 **자세한 값이 나오진 않게 된다**.

다음은 1달러로 0.1달러 상품을 10번 구입하고 남은 잔돈을 계산하는 코드이다.

```JAVA
public static void main(String[] args) {
    double budget = 1.00;
    double price = 0.10;
    int count = 10;
    
    double change = getChange(budget, price, count);
    System.out.println("잔돈: $ " + change);
}

private static double getChange(double budget, double price, int count) {
    return budget - (price * count)
}
```

코드를 실행해보면 잔돈이 $0 남았을 것 같지만, 실제 실행해보면 $0.0999999... 남아있는 기적의 계산법이 눈 앞에 펼쳐진다.

결과를 출력하기 전에 반올림하는 방법도 통하지 않는다.

다음 코드는 $0.1, 0.2, 0.3 ... 1 가치의 상품을 $1로 차례대로 산다고 했을 때, 몇 개를 사고 잔돈이 얼마나 남는지 계산하는 코드다. 
```JAVA
public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈: " + funds);
}
```

원래라면 0.1 ~ 0.4 까지 4개를 구매하고 딱 $0를 남겨야겠지만,

코드를 실행해보면, 앞의 3개를 구입하고 남은 돈은 $0.3999...다. 

### BigDecimal & int, long

자바는 이런 오류를 해결하기 위해 `BigDecimal` 이라는 클래스를 제공한다.

위의 코드를 `BigDecimal`을 사용해 바꿔보자.

```JAVA
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS;
         funds.compareTo(price) >= 0;
         price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈: " + funds);
}
```

이러면 드디어 4개를 구입하고 잔돈 $0 가 남는다.

하지만 `BigDecimal`은 다른 기본 타입 수처럼 사칙연산 기호를 사용해 직관적인 연산을 할 수 없어 `.add()` 같은 메서드 호출로만 연산이 가능하고, 무엇보다 일단 계산보다 훨씬 느리다는 단점들이 있다.

이런 단점들을 피하고 싶다면, `int` 나 `long` 타입을 사용하는 것으로 우회할 수는 있다.

두 타입을 사용한다면, 소수점 관리는 직접 해야 한다는 불편함은 감수해야 한다.

```JAVA
public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈: " + funds);
}
```