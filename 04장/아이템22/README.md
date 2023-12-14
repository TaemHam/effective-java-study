# 아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 오직 "클라이언트가 이 인스턴스로 무엇을 할 수 있는지 알려주는 용도"여야 한다.

상수를 모아놓고 관리하는 목적으로 인터페이스에 static final 필드만 넣어놓고, 이 상수를 사용하기 위해 그 인터페이스를 implements로 넣는 경우가 있다. 결론적으로 말하자면, **이렇게 사용하는 것은 인터페이스를 잘못 사용한 것**이다.

```JAVA
// 안티패턴 상수 인터페이스
public interface PhysicalConstants{
      static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      static final double ELECTRON_MASS = 9.109_383_56e-3;
}
```

### 왜 상수 인터페이스는 잘못된걸까?

1. 클래스 내부에서 사용하는 상수는 외부에서 참조할 필요 없는 **내부 구현**의 영역이다.

    하지만 인터페이스를 구현하면 이 상수들이 외부에 노출되게 되고, 해당 클래스를 사용하는 다른 개발자에게 혼란을 일으킬 수 있다.

2. 클래스에 이 상수들이 종속되게 된다.

    다음 릴리즈에서 이 상수들이 쓰이지 않더라도, 바이너리 호환성을 위해 인터페이스를 뺄 수가 없게 된다.

> **📌 참고**<br><br>
> 호환성의 종류
> * 바이너리 호환성 : 다시 컴파일 하지 않은 기존 바이너리 코드로 최신 API를 실행할 수 있는 상황.
> * 소스 호환성 : 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있는 상황.
> * 동작 호환성 : 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행하는 상황.

3. 하위 클래스가 있다면, 이름 공간이 사용하지 않는 상수들로 오염되어 버린다.

### 상수 사용을 위한 대안은 무엇이 있을까?

1. 클래스나 인터페이스 자체에 추가한다.

2. 열거타입으로 만들어 공개한다.

3. 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개한다.

```JAVA
public class PysicalConstants{
      private PysicalConstants(){}; // 인스턴스화 방지
      public static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      public static final double ELECTRON_MASS = 9.109_383_56e-3;
}
```