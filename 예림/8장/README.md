# 8장. 데이터 흐름
> 자바 8의 등장으로 일반적인 자바 프로그램 내에 람다 표현식과 스트림으로 함수형 프로그래밍 패러다임을 바로 넣을수 있게 됨
> 

## 8.1 익명 클래스 대신 람다 사용하기

### 개선 전 코드

```java
class Calculator {

    Map<Double, Double> values = new HashMap<>();

    Double square(Double x) {
        Function<Double, Double> squareFunction =
                new Function<Double, Double>() {
                    @Override
                    public Double apply(Double value) {
                        return value * value;
                    }
                };
        return values.computeIfAbsent(x, squareFunction);
    }
}
```

- 자바 8에서 등장한 computeIfAbsent() 메서드는 키를 이용해 맵에서 값을 얻는데 맵에 키가 없으면 값을 먼저 계산함
    - 하지만 메서드를 사용하려면 맵에 키가 존재하지 않을 때 어떻게 값을 계산할 것인지에 대한 로직을 입력 매개변수로 제공해야 함
    - 타입 관점에서 computeIfAbsent()에는 Function<Double, Double> 인터페이스를 구현하면서 Double apply(Double value) 메서드를 포함하는 클래스의 인스턴스가 필요함
    - Double이라는 타입은 values 맵에 들어있는 타입에 기반해 정한 것이므로 맵이 다르면 타입도 달라짐
- 위 코드에서 프로그래머는 인터페이스를 구현할 익명 클래스를 초기화함 (클래스명이 없고 클래스에 인스턴스만 있어서 익명이라고 부름)
- 하지만 익명 클래스를 사용하면 코드가 매우 커지고 들여쓰기 수준도 높아짐
- **람다 표현식으로 코드 향상 가능**

### 개선 후 코드

```java
class Calculator {

    Map<Double, Double> values = new HashMap<>();

    Double square(Double value) {
        Function<Double, Double> squareFunction = factor -> factor * factor;
        return values.computeIfAbsent(value, squareFunction);
    }
}
```

- 람다는 함수형 인터페이스, 즉 단일 추상 메서드를 포함하는 인터페이스를 구현함
- 람다는 한 줄이나 여러 줄로 작성할 수 있음

```java
// one-liner
Function<Double, Double> squareFunction = factor -> factor * factor;
//  multi-liner
Function<Double, Double> squareFunction = factor -> {
    return factor * factor;
};
```

- 일반적으로 자바에서는 어떤 경우든 타입을 명시적으로 표기해야 하지만 람다 표현식의 매개변수라면 대부분의 경우 컴파일러 스스로 타입을 알아낼 수 있음
- 람다 표현식에서 유일하게 구현하고 있는 추상 메서드를 찾아내 메서드 서명을 타입 명세로 사용하면 됨 (타입 추론)

## 8.2 명령형 방식 대신 함수형

- 컬렉션 처리에 대해서라면 함수형 프로그래밍 방식이 명령형 방식보다 훨씬 읽기 쉬움

### 개선 전 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        **List<String> names = new ArrayList<>();**
        for (Supply supply : supplies) {
            if (supply.isUncontaminated()) {
                String name = supply.getName();
                if (!names.contains(name)) {
                    names.add(name);
                }
            }
        }
        return names.size();
    }
}
```

- 위 코드는 명령형 방식으로 작성되었고 매우 흔한 작업을 수행하는 짧지만 매우 복잡한 메서드가 나옴
    - 무엇을 해야 하는지, 루프와 조건, 변수 할당문, 메서드 호출로 어떻게 해야 하는지 컴퓨터에게 명령함
- 하지만 일반적으로 코드가 **무엇을** 하는지에 관심이 있지 목표에 **어떻게** 도달하는지 별 관심이 없음
- 람다 표현식으로는 **무엇**이 이루어지길 원하는지 명시할 수 있을 뿐 **어떻게**는 명시할 수 없음 → 간결한 코드가 됨

### 개선 후 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        **return supplies.stream()**
                       .filter(supply -> supply.isUncontaminated())
                       .map(supply -> supply.getName())
                       .distinct()
                       .count();
    }
}
```

- stream()은 컬렉션을 스트림으로 변환
- filter()로 오염되지 않은 supply만 남김
- 타입(Supply)을 다른 타입(String)으로 매핑
- distinct()로 중복은 버림
- count()로 스트림 내 남은 개수를 셈. count()는 스트림을 끝내고 명령형 방식으로 다시 돌려보내는 종료 연산자임
