# 아이템 23. 태그 달린 클래스보다는 클래스 계층주고를 활용하라

### 태그 클래스란?

클래스의 인스턴스에 태그를 붙여 인스턴스가 가지는 의미을 표현하도록 한 클래스

```JAVA
// 도형을 표현하는 클래스
class Figure {

    // 태그로 가질 수 있는 값
	enum Shape { RECTANGLE, CIRCLE };

	// 태그 필드 - 현재 도형의 모양을 의미
	private final Shape shape;

	// 직사각형일 때 사용하는 필드
	private double length;
	private double width;
	// 원일 때 사용하는 필드
	private double radius;

	// 직사각형일 때 호출하는 생성자
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}

	// 원일 때 호출하는 생성자
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	double area() {
		switch(shape) {
			case RECTANGLE:
			return length * width;

			case CIRCLE:
			return Math.PI * (radius * radius);

			default:
			throw new AssertionError(shape);
		}
	}
}
```

### 왜 태그 클래스를 쓰면 안될까?

1. 열거 타입 선언, 태그 필드, switch문 등, 
2. 여러 구현이 한 클래스에 혼합돼 있어서 가독성이 나쁘다.
3. 다른 모양을 가질때를 위한 코드도 같이 가지고 있어, 메모리를 많이 사용한다.
4. 필드들을 final로 사용하려면 쓰이지 않는 필드까지 생성자에서 초기화 해야한다.
5. 또 다른 의미를 추가하려면 코드 전체를 수정해야 한다. (모든 switch 분기에 하나를 추가해야 한다)
6. 인스턴스의 타입만으로는 현재 어떤 의미를 가지는지 알 수 없다.

**즉, 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.**

### 그럼 어떤 방법이 더 좋을까?

클래스 계층구조를 사용한 `서브타이핑(Subtyping)`이다.

### 어떻게 서브타이핑으로 바꿀까?

1. 계층 구조의 루트가 될 추상 클래스를 정의한다.

  * 태그 값에 따라 동작이 달라 지는 메서드들을 추상 메서드로 선언한다.

  * 태그에 상관없이 동일하게 동작하는 메서드는 일반 메서드로, 공통으로 사용하는 필드는 일반 필드로 올린다.

```JAVA
abstract class Figure {
    abstract double area();
}
```

2. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.

```JAVA
class Circle extends Figure {

    private final double radius;

    Circle(double radius) { 
        this.radius = radius; 
    }

    @Override 
    double area() { 
        return Math.PI * (radius * radius); 
    }
}

class Rectangle extends Figure {

    private final double length;
    private final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    @Override double area() { 
        return length * width; 
    }
}
```

### 서브타이핑의 장점

* 타입이 의미 별로 따로 존재하니 변수의 의미를 명시하거나 제한할 수 있다.
* 메서드에서 사용할 때 특정 의미의 클래스만 매개 변수로 받도록 제한할 수도 있다. 
* 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일 타임 타입 검사 능력을 높여줄 수 있다.