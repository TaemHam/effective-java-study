# 아이템 63. 문자열 연결은 느리니 주의하라

## 문자열 연결을 많이 할 때 주의하자

- 자바에서 + 연산자는 문자열을 연결해주는 편리한 수단이다.

- 그러나 문자열 연결 연산자로 문자열 n개를 연결하는 시간은 n^2이기 때문에 주의해야한다.
  - 왜냐하면 문자열은 불변이므로 두 문자열을 연결하려면 두 개 모두 복사해야 하기 때문이다.

<br>

## 문자열 연결이 많을 때는 StringBuilder를 쓰자.

```java
public static void main(String[] args) {
    List<String> list = getList();

    long time1 = useOperator(list);
    long time2 = useStringBuilder(list);

    System.out.println("time1 = " + time1);
    System.out.println("time2 = " + time2);
}

private static long useOperator(List<String> list) {
    String result = "";

    long start = System.currentTimeMillis();

    for (String elem : list) {
        result += elem;
    }

    long end = System.currentTimeMillis();

    return end - start;
}

private static long useStringBuilder(List<String> list) {
    StringBuilder sb = new StringBuilder();

    long start = System.currentTimeMillis();

    for (String elem : list) {
        sb.append(elem);
    }

    long end = System.currentTimeMillis();

    return end - start;
}

private static List<String> getList() {
    List<String> list = new ArrayList<>();

    for (int i = 0; i < 100; i++) {
        list.add(String.valueOf(i));
    }

    return list;
}
```

- 위의 코드는 문자열 100개를 연결할 때, 문자열 연산자를 사용하는 방법과 StringBuilder를 사용하는 방법의 소요 시간을 비교한 것이다.

- 실행해보면 문자열 100개만 연결해도 시간차이가 나는 것을 알 수 있다.
