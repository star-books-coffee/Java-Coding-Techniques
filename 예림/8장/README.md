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

## 8.3 람다 대신 메서드 참조

### 개선 전 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        return supplies.stream()
                       .filter(supply -> !supply.isContaminated())
                       .map(supply -> supply.getName())
                       .distinct()
                       .count();
    }
}
```

- 람다 표현식은 스트림 중간부터 실행할 수 없으며 오직 스트림 전체에 대해서만 실행 가능
- 람다 표현식의 일부만 테스트하기 어려움
- 표현식을 변수로 참조하지 않다보니 다른 어디에도 사용할 수 없음 → 람다 표현식은 자신을 감싼 메서드에만 귀속됨
- 람다 표현식에 논리가 더 들어가면 잠재적인 오류 가능성이 있음
- 게다가 람다 표현식은 참조가 불가능해 단위 테스트를 통해 별개로 테스트할 수가 없으니 기대대로 작동하는지 검증하기 어려움
- 반면 메서드 호출에 기반한 코드는 테스트하기 쉬움
- 자바의 함수형 프로그래밍에도 이것을 처리할 **메서드 참조** 메커니즘이 있음

### 개선 후 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        return supplies.stream()
                       .filter(Supply::isUncontaminated)
                       .map(Supply::getName)
                       .distinct()
                       .count();
    }
}
```

- 람다 표현식을 정의하는 대신 미리 정의된 메서드를 스트림 내에서 바로 참조함
- 스트림은 미리 정의된(물론 테스트까지 끝난) 메서드를 조합할 뿐임
- 메서드 참조에는 특수한 `ClassName::methodName` 형식의 문법을 써야 함
- 필터 연산에는 Predicate 인터페이스에 맞는 메서드 참조(객체를 받아 불을 반환하는 메서드), 맵 연산에는 Function 메서드에 맞는 메서드 참조(객체를 받아 객체를 반환하는 메서드)를 써야 함

## 8.4 부수 효과 피하기

### 개선 전 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        List<String> names = new ArrayList<>();

        **Consumer<String> addToNames = name -> names.add(name);**

        supplies.stream()
                .filter(Supply::isUncontaminated)
                .map(Supply::getName)
                .distinct()
                .forEach(**addToNames**);
        return names.size();
    }
}
```

- 이론상 함수형 프로그래밍에는 부수 효과(side effect)가 없다
    - 모두 입력으로 데이터를 받아 출력으로 새로운 데이터를 생성하는 함수 뿐임
    - 오가는 데이터는 불변
- 위 예제는 목표를 이루기 위해 부수 효과에 크게 의존하고 있음
- `Consumer addToNames`에서 Consumer는 람다 표현식 밖에 있는 리스트에 원소를 추가함
    
    ⇒ 부수효과 발생
    
- 코드에 기능상 오류는 없지만 동시 실행이 가능하도록 바꾸면 쉽게 고장이 남
- 종종 명령형 방식으로 돌아가 스트림을 종료시키려고 하는데 이것은 부수효과를 통해서만 가능함
- 부수효과를 일으키지 않으면서 람다 표현식을 종료하는 더 좋은 방법이 없을까?

### 개선 후 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        List<String> names = supplies.stream()
                                     .filter(Supply::isUncontaminated)
                                     .map(Supply::getName)
                                     .distinct()
                                     **.collect(Collectors.toList());**
        return names.size();
    }
}
```

- 리스트를 직접 만들지 않고 컬렉션 내 스트림에 남아있는 각 원소를 collect() 함
- 리스트를 만들려면 `collect(Collectors.toList())`로 스트림을 종료해야 함
- 예제의 최종 목표는 결과 리스트의 크기이므로 다음과 같이 작성할 수 있음

```java
long countDifferentKinds() {
    return supplies.stream()
                   .filter(Supply::isUncontaminated)
                   .map(Supply::getName)
                   .distinct()
                   .count();
}
```

- `count()`는 Stream 클래스의 `reduce()` 연산자, 즉 `reduce(0, (currentResult, streamElement) → currentResult + 1)`의 단축형임
- 스트림을 종료시킬 때 ForEach()는 쉽게 부수 효과를 일으키니 가능하면 쓰지 말자

## 8.5 복잡한 스트림 종료 시 컬렉트 사용하기

### 개선 전 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    Map<String, Long> countDifferentKinds() {
        Map<String, Long> nameToCount = new HashMap<>();

        Consumer<String> addToNames = name -> {
            if (!nameToCount.containsKey(name)) {
                nameToCount.put(name, 0L);
            }
            nameToCount.put(name, nameToCount.get(name) + 1);
        };

        supplies.stream()
                .filter(Supply::isUncontaminated)
                .map(Supply::getName)
                .forEach(addToNames);
        return nameToCount;
    }
}
```

- `reduce()` 연산자는 원시값보다 복잡한 값이 결과로 나오는 스트림을 종료시킬 때 가장 적합함
- 위 코드는 `supplies` 리스트 내 고유 원소 수를 모두 세는 대신 남은 제품 수를 이름별로 묶어 계산해 `Map<String, long>` 형태로 만듦
- `Map<String, long> nameToCount`를 계산할 때 부수효과에 크게 의존함

### 개선 후 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    Map<String, Long> countDifferentKinds() {
        return supplies.stream()
                       .filter(Supply::isUncontaminated)
                       **.collect(Collectors.groupingBy(Supply::getName,
                               Collectors.counting())
                       );**
    }
}
```

- 미리 정의된 여러 Collectors를 제공
- Collectors.groupingBy() 연산자를 Supply 인스턴스의 스트림에 적용
    - 이 연산자는 항상 Map 구조를 반환함
- groupingBy() 호출의 두 번째 매개변수인 Collectors.counting()은 한 그룹 내 Supply 인스턴스 수를 셈

## 8.6 스트림 내 예외 피하기

### 개선 전 코드

```java
class LogBooks {

    static List<LogBook> getAll() throws IOException {
        **return Files.walk(Paths.get("/var/log"))**
                    .filter(Files::isRegularFile)
                    .filter(LogBook::isLogbook)
                    .map(path -> {
                        try {
                            return new LogBook(path);
                        } catch (IOException e) {
                            **throw new UncheckedIOException(e);**
                        }
                    })
                    .collect(Collectors.toList());
    }
}
```

- 파일 시스템을 다룰 때는 항상 IOException이 일어날 가능성을 염두에 두어야 함
- 문제는 스트림에 검증된 예외를 쓸 수 없다는 점임. 반드시 스트림 내에서 예외를 처리해야 함
- 자바의 함수형 방식에는 예외, 심지어 검증되지 않은 예외까지 처리할 적절한 메커니즘이 전혀 없음
- 함수는 입력을 처리해 출력을 생성할 뿐임. 예외를 던지거나 잡지 않음
- 함수형 프로그래밍 방식에서 벗어나지 않으면서 예외를 어떻게 처리할까?

### 개선 후 코드

```java
lass LogBooks {

    static List<LogBook> getAll() throws IOException {
        **try (Stream<Path> stream = Files.walk(Paths.get("/var/log"))) {**
            return stream.filter(Files::isRegularFile)
                         .filter(LogBook::isLogbook)
                         **.flatMap**(path -> {
                             try {
                                 return Stream.of(new LogBook(path));
                             } catch (IOException e) {
                                 **return Stream.empty();**
                             }
                         })
                         .collect(Collectors.toList());
        }
    }
}
```

- 검증된 예외를 검증되지 않은 예외로 더 이상 반환하지 않고 그 대신 스트림에서 예외 원소를 간단히 제거함
- 이것을 위해 flatMap() 연산자를 사용함
    - 어떤 타입을 다른 타입으로 매핑하는 대신 어떤 타입을 다른 타입의 Stream으로 매핑함
    - 제대로 동작하면 Stream.of(element)가 수행되어 원소 하나가 포함된 새로운 스트림을 생성
- 어떤 일이 발생하든 예외는 전체 연산을 중지시키지 않으며 스트림은 입력에 따른 출력을 생성함
- 예외를 피하려면 함수형 방식을 따르는 편이 나음

## 8.7 널 대신 옵셔녈

### 개선 전 코드

```java
class Communicator {

    Connection connectionToEarth;

    void establishConnection() {
        // connectionToEarth를 할당하는 데 쓰이지만 불안정할 수 있음
    }

    Connection getConnectionToEarth() {
        return **connectionToEarth;**
    }
}
```

- 참조에 어떻게 접근하는지 완벽히 제어할 수 있는 내부 상태라면 null 참조를 사용해도 좋음
- 위 코드에 나오는 Connection은 끊겨 있을 수도 있고 connectionToEarth이 null일 수도 있음
- 호출하는 코드에서 null을 검증하지 않으면 문제가 발생할 수 있음

### 개선 후 코드

```java
class Communicator {

    Connection connectionToEarth;

    void establishConnection() {
        // connectionToEarth를 할당하는 데 쓰이지만 불안정할 수 있음
    }

    Optional<Connection> getConnectionToEarth() {
        **return Optional.ofNullable(connectionToEarth);**
    }
}
```

- `Optional`은 있을 수도 있고 없을 수도 있는 객체를 위한 임시 저장소임
- 객체나 null을 가리킬지도 모를 참조를 넣어 `Optional.ofNullable()`을 호출해 생성함
- 이제 용법을 살펴보자
    
    ```java
    Connection connection = communicator.getConnectionToEarth()
                                        .orElse(null);
    connection.send("Houston, we got a problem!");
    ```
    
    - 위 용법(orElse(null))처럼 사용하면 여전히 예외가 발생함
    
    ```java
    communicationSystem.getConnectionToEarth()
    				            .ifPresent(connection ->
    				                connection.send("Houston, we got a problem!")
    				            );
    ```
    
    - ifPresent()는 Optional 내 값이 null이 아닐 때만 전달한 Consumer를 실행함

## 8.8 선택 필드나 매개변수 피하기

### 개선 전 코드

```java
class Communicator {

    Optional<Connection> connectionToEarth;
    
    void setConnectionToEarth(Optional<Connection> connectionToEarth) {
        this.connectionToEarth = connectionToEarth;
    }
    Optional<Connection> getConnectionToEarth() {
        return connectionToEarth;
    }
}
```

- Optional에는 부재(Optional.empty()) 또는 존재라는 두 가지 상태가 있음
- 하지만 옵셔널 필드나 메서드 매개변수라면 변수가 null일 수도 있음 → 상태가 3개로 늘어남
- 필드가 매개변수에 옵셔널 타입을 사용하면 더 복잡해질 뿐임

### 개선 후 코드

```java
class Communicator {

    **Connection connectionToEarth;**

    void setConnectionToEarth(Connection connectionToEarth) {
        this.connectionToEarth = Objects.requireNonNull(connectionToEarth);
    }
    **Optional<Connection> getConnectionToEarth() {
        return Optional.ofNullable(connectionToEarth);
    }**

    void reset() {
        connectionToEarth = null;
    }
}
```

- 필드와 메서드 매개변수 타입에서 Optional 부분을 제거해야 함
- Optional 타입은 반환값에만 써야 함
- Objects.requiredNull()이라는 메서드로 null 값을 검증함
- 모든 null 참조는 필드 접근을 완벽히 제어할 수 있는 클래스 내부에서만 발생함

## 8.9 옵셔널을 스트림으로 사용하기

### 개선 전 코드

```java
class BackupJob {

    Communicator communicator;
    Storage storage;

    void backupToEarth() {
        **Optional<Connection>** connectionOptional =
                communicator.getConnectionToEarth();
        if (!connectionOptional.isPresent()) {
            throw new IllegalStateException();
        }

        **Connection connection = connectionOptional.get();**
        if (!connection.isFree()) {
            throw new IllegalStateException();
        }

        connection.send(storage.getBackup());
    }
}
```

- Optional은 0개 또는 1개 원소만 포함하는 특별한 형식의 스트림임
- filter()나 map()과 같은 일반적인 스트림 연산을 모두 Optional에 바로 적용할 수 있음
- 위 예제에서 Optional을 이상한 이름의 변수에 저장한 이유는 임피던스 불일치 때문임
- 명령형 방식과 함수형 방식 간에 변환하려면 get()과 같은 메서드로 변환을 수행해야 함
- 이러한 문맥 교환만 없어도 코드는 훨씬 읽기 쉬워짐

### 개선 전 코드

```java
class BackupJob {

    Communicator communicator;
    Storage storage;

    void backupToEarth() {
        Connection connection = communicator.getConnectionToEarth()
                .filter(Connection::isFree)
                .orElseThrow(IllegalStateException::new);
        connection.send(storage.getBackup());
    }
}
```

- orElseThrow() 연산을 통해 연결이 끊겼거나 사용할 수 없을 때 무슨 일이 일어나는지까지 분명히 알려줌
- 다른 상황에서는 다른 Optional 메서드가 적절할 수도 있음
- 예를 들어 정말 값이 부재여도 상관 없다면 그냥 ifPresent()만 호출함
- 만약 Optional을 읽기만 해도 된다면 Optional을 반환해 .map().orElse(defaultValue)로 기본값을 제공함
- 두 번째 용법이 더 흔함
    
    ```java
    String state = communicator.getConnectionToEarth()
                              .map(Connection::isFree)
                              .map(isFree -> isFree ? "free" : "busy")
                              .orElse("absent");
    ```