# 아이템 20. 추상 클래스보다는 인터페이스를 우선하라.

자바 8부터 인터페이스도 디폴트 메서드를 제공할 수 있게 되어서, 추상 클래스와 인터페이스 모두 메서드를 구현된 채로 제공할 수 있게 되었다.

### 왜 인터페이스가 더 우선일까?

1. 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.

    * 인터페이스는 새로 만들어 추가 메서드를 넣고, 구현할 클래스에 더 추가하면 끝이다.

    * 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어렵다. 두 클래스가 같은 추상 클래스를 확장하길 원한다면, 그 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야 한다. 이 방식은 그렇게 하는 것이 적절하지 않은 상황에서도 새로 추가된 추상 클래스의 모든 자손이 이를 상속하게 된다.

2. 인터페이스는 믹스인(Mixin) 정의에 안성맞춤이다.

    `믹스인`이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 기능을 제공한다고 선언하는 효과를 준다. 

    * 인터페이스는 믹스인을 정의하는 데 딱 알맞다. 대상 클래스의 논리적 순서를 정해주는 `Comparable` 인터페이스가 이런 것이다.

    * 추상 클래스는 기존 클래스에 덧씌울 수 없어서 믹스인을 정의할 수 없다. 클래스는 두 부모를 섬길 수 없고, 클래스 계층구조에는 믹스인을 삽입하기에 합리적인 위치가 없기 때문이다.

3. 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

    쉽게 말해, 클래스를 여러 타입으로 사용하고 싶을 때 상하관계가 명확한 계층구조를 사용하지 않아도 된다는 말이다. 

    * 클래스는 여러 인터페이스를 구현할 수 있기 때문에, 계층구조로 표현하지 않아도 여러 타입으로 표현 가능하다. 예를 들어, `Singer` 인터페이스와 `Songwriter` 인터페이스가 있을 때, 두 인터페이스 모두를 구현한 `SingerSongwriter` 클래스를 정의할 수 있다. 이 클래스는 Singer나 Songwriter 타입으로도 쓸 수 있다.

    * 추상 클래스도 계층 구조를 사용한다면 위처럼 여러 타입으로 사용할 수는 있다. 하지만 만약 선택 가능한 인터페이스가 두 개보다 많다면, 원하는 조합마다 새로운 추상 클래스로 정의해야 하는, 이른바 조합 폭발(combinatorial explosion) 현상이 일어난다.

4. 래퍼 클래스와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다. 

  -- 이 부분 논의 필요

### 디폴트 메서드란?

디폴트 메서드란, 인터페이스의 메서드 중 구현 방법이 명백한 것을 구현해놓은 메서드를 의미한다. 이는 프로그래머의 일을 상당히 덜어준다.

디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 `@implSpec` 자바독 태그를 붙여 문서화해야 한다.

### 디폴트 메서드의 제약

많은 인터페이스가 equals와 hashCode 같은 Object의 메서드를 정의하고 있지만, 이들은 디폴트 메서드로 제공해서는 안 된다. 

또한 인터페이스는 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버도 가질 수 없다(단, private 정적 메서드는 예외다).

### 인터페이스의 장점과 추상 클래스의 장점을 합친 건 없을까?

예를 들어, 두 개의 클래스들을 같은 타입으로 담고싶기도 하고, 내부의 메서드들 중에서도 중복 코드를 가진 메서드의 중복을 없애고 싶다면 어떻게 해야할까?

이는 `추상 골격 구현`(Skeletal Implementation)로 해결할 수 있다. 
추상 골격 구현은 다음 순서로 정의할 수 있다.

1. 중복 코드를 가지는 메서드를 추려낸다.

(인터페이스를 수정할 수 있는 상황)

2. 인터페이스에 해당 코드를 디폴트 메서드로 수정한다.

(인터페이스를 수정할 수 없는 상황)

2. 인터페이스를 구현하는 추상 클래스를 만들고, 중복 코드를 넣는다.

3. 인터페이스를 구현하는 새로운 구현 클래스도 만들고, 내부에 중첩 클래스로 추상 클래스를 상속하는 클래스를 만들어 나머지 추상 메서드들을 구현한다.

4. 중첩 클래스의 인스턴스를 구현 클래스의 내부 필드로 담아 두고, 인터페이스로 구현할 메서드에서 중첩 클래스의 메서드들을 호출해 로직 처리를 맡긴다.

<details>
<summary>예제 코드 보기</summary>

[[출처](https://dzone.com/articles/favour-skeletal-interface-in-java)]

``` JAVA

// 구현할 인터페이스
public interface Ivending {
    // 얘네 셋은 동일 로직
    void start();
    void chooseProduct();
    void stop();

    // 얘는 다른 로직
    void process();
}

---------

// 골격 클래스로 추상 클래스 작성
public abstract class AbstractVending implements Ivending {

    // 동일 로직들은 여기에 작성
    public void start() {
        System.out.println("Start Vending machine");
    }

    public void stop() {
        System.out.println("Stop Vending machine");
    }

    public void process() {
        start();
        chooseProduct();
        stop();
    }

    // chooseProduct는 아직 구현 안된 상태.
}

---------

// 상속할 클래스가 없어서 추상 클래스를 상속할 수 있는 경우
public class CandyVending extends AbstractVending implements Ivending { 

    @Override
    public void chooseProduct() {
        System.out.println("Produce diiferent candies");
        System.out.println("Choose a type of candy");
        System.out.println("pay for candy");
        System.out.println("collect candy");
    }
}

---------

// 다음 설명할 클래스를 위한 상속용 클래스
public class VendingService {
    public void service() {
        System.out.println("Clean the vending machine");
    }
}

---------

// 상속할 클래스가 이미 있어 추상 클래스를 상속할 수 없는 경우
public class DrinkVending extends VendingService implements Ivending { 

    // 중첩 클래스로 두고, 추상 메서드 구현
    private class AbstractVendingDelegator extends AbstractVending {
        @Override
        public void chooseProduct() {
            System.out.println("Produce diiferent soft drinks");
            System.out.println("Choose a type of soft drinks");
            System.out.println("pay for drinks");
            System.out.println("collect drinks");
        }
    }

    // 위 중첩 클래스 인스턴스를 필드에 두고, 모든 메서드 그대로 위임
    AbstractVendingDelegator delegator = new AbstractVendingDelegator();

    @Override
    public void start() {
        delegator.start();
    }

    @Override
    public void chooseProduct() {
        delegator.chooseProduct();
    }

    @Override
    public void stop() {
        delegator.stop();
    }

    @Override
    public void process() {
        delegator.process();
    }
}

```

</details>

