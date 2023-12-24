## 8장. 데이터 흐름
### 8.1. 익명 클래스 대신 람다 사용하기

- `computeIfAbsent()`
    - key 존재하는 경우 : 기존에 존재하는 key 의 value 값 리턴
    - key 존재하지 않는 경우 : 람다식을 적용한 값을 해당 key 에 저장하고 newValue 반환
    
    ```java
    // 이전
    Map<Key, Value> map = new HashMap();
    
    Value value = map.get(key);
    if (value == null) {
        value = getNewValue(key);
        map.put(key, value);
    }
    
    // 이후
    Map<Key, Value> map = new HashMap();
    Value value = map.computeIfAbsent(key, k -> getNewValue(key));
    ```
    
    - 메서드 사용하려면 맵에 키 존재하지 않을 때 `어떻게` 값을 계산할 건지에 대한 로직을 입력 매개변수로 제공해야 함
- 개선 전 코드
    
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
    
    - computeIfAbsent() 는 Function<Double, Double> 인터페이스 구현 + Double apply(Double value) 메서드를 포함하는 클래스 인스턴스가 필요
        
        → 따라서 인터페이스를 구현하는 `익명클래스` 를 만들어서 사용 (클래스명 없고, 클래스에 인스턴스가 딱 1개라서 익명임)
        
- computeIfAbsent 내부 동작
    
    ```java
    @Override
    public V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {
    	...
    	V v = mappingFunction.apply(key);
    	...
    }
    ```
    
- Function<T,R> 함수형 인터페이스 명세
    
    ```java
    @FunctionalInterface
    public interface Function<T, R> {
    
        /**
         * Applies this function to the given argument.
         *
         * @param t the function argument
         * @return the function result
         */
        R apply(T t); // 인수와 반환 타입이 다른 함수
    ```
    
- 개선 후 코드
    
    ```java
    class Calculator {
    
        Map<Double, Double> values = new HashMap<>();
    
        Double square(Double value) {
            Function<Double, Double> squareFunction = factor -> factor * factor;
            return values.computeIfAbsent(value, squareFunction);
        }
    }
    ```
    
    - 람다는 함수형 인터페이스 ; 즉, 단일 추상 메서드를 포함하는 인터페이스를 구현
        - Function 인터페이스가 apply() 라는 추상메서드 하나를 포함하니 람다와 딱 들어맞음!
    - 람다 표현식의 매개변수는, 컴파일러 스스로 타입을 알아낼 수 있음
    - 람다표현식에서 유일하게 구현하고 있는 **추상메서드를 찾아, 메서드 내 서명을 타입 명세로 사용**하면 됨 → `타입 추론`
- **익명클래스 내의 추상메서드를 override 하지 말고 람다식을 사용하여 표현하라**

### 8.2. 명령형 방식 대신 함수형

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        List<String> names = new ArrayList<>();
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

- 명령형 코드 : 무엇을 해야 하는지, 루프와 조건, 변수 할당, 메서드 호출로 `어떻게` 해야 하는지 명령함
- 일반적으로 코드는 **무엇을 하는지에 관심이 있지**, 어떻게 목표에 도달하는지는 관심 없음

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        return supplies.stream() // for 문 대체
                       .filter(supply -> supply.isUncontaminated()) // if 문 대체
                       .map(supply -> supply.getName()) // 변수에 담는 코드 대체
                       .distinct() // 포함여부 판별하는 if 문 대체
                       .count(); // size() 대체
    }
}
```

- **람다표현식을 사용하여 어떻게 하는지가 아니라 무엇을 하는지만 명시하자**
- filter 메서드 명세
    
    ```java
    Stream<T> filter(Predicate<? super T> predicate);
    
    @FunctionalInterface
    public interface Predicate<T> {
    
        /**
         * Evaluates this predicate on the given argument.
         *
         * @param t the input argument
         * @return {@code true} if the input argument matches the predicate,
         * otherwise {@code false}
         */
        boolean test(T t);
    ```
    
- map 메서드 명세
    
    ```java
    <R> Stream<R> map(Function<? super T, ? extends R> mapper);
    
    @FunctionalInterface
    public interface Function<T, R> {
    
        /**
         * Applies this function to the given argument.
         *
         * @param t the function argument
         * @return the function result
         */
        R apply(T t);
    ```
    

### 8.3. 람다 대신 메서드 참조

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

- 람다 표현식은 스트림 중간부터 실행할 수 없고, **오직 스트림 전체에 대해서만** 실행가능하여 람다 표현식 일부만 테스트하기 어려움
- 위 코드는 표현식을 변수로 참조하지 않다보니 다른 어디에서도 사용할 수 없고, **자신을 감싼 메서드내에만 귀속됨**
- 이러한 특징은 표현식이 복잡해지면 잠재적 오류 가능성이 있는데, 참조가 불가능해서 단위 테스트로 별개 테스트도 불가능하므로 검증하기 어려움

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

- `메서드 참조` 를 이용하면 메서드 호출에 기반한 코드이므로 테스트 하기 쉬움
    - 람다 표현식 정의하는 대신, **미리 정의된 메서드를 스트림 내에서 바로 참조하자**
    - 스트림은 미리 정의된 (테스트도 완료된) 메서드를 조합할 뿐

### 8.4. 부수 효과 피하기

- 이론상 함수형 프로그래밍은 부수효과가 없음 ; 즉, 입력으로 데이터를 받아 출력으로 새로운 데이터를 생성하는 함수일 뿐 + 오가는 데이터는 불변

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        List<String> names = new ArrayList<>();

        Consumer<String> addToNames = name -> names.add(name);

        supplies.stream()
                .filter(Supply::isUncontaminated)
                .map(Supply::getName)
                .distinct()
                .forEach(addToNames);
        return names.size();
    }
}
```

- 위 코드에서 Consumer 는 람다 표현식 밖의 list 에 원소를 추가함으로써 부수효과가 발생함
    - 코드 기능상 오류는 없지만, 동시 실행이 가능하도록 바꾸면 쉽게 오류가 발생할 수 있음
- consumer 인터페이스 명세
    
    ```java
    @FunctionalInterface
    public interface Consumer<T> {
    
        /**
         * Performs this operation on the given argument.
         *
         * @param t the input argument
         */
        void accept(T t); // 한 개의 인수를 받고 반환값이 없는 함수
    ```
    

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    long countDifferentKinds() {
        List<String> names = supplies.stream()
                                     .filter(Supply::isUncontaminated)
                                     .map(Supply::getName)
                                     .distinct()
                                     .collect(Collectors.toList());
        return names.size();
    }
}
```

- 리스트를 직접 만들지 말고, 컬렉션 내 스트림에 남아있는 각 원소를 collect() 하자
- **스트림을 종료시킬 때, forEach 는 쉽게 부수효과를 일으키니 사용하지 말고 collect() 나 reduce() 를 사용하자**

### 8.5. 복잡한 스트림 종료 시 컬렉트 사용하기

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

- 위의 코드는 forEach 문에서 부수효과를 일으키며, 코드도 읽기 어려움

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    Map<String, Long> countDifferentKinds() {
        return supplies.stream()
                       .filter(Supply::isUncontaminated)
                       .collect(Collectors.groupingBy(Supply::getName,
                               Collectors.counting())
                       );
    }
}
```

- `Collectors.groupingBy()` : 항상 Map 자료구조 반환
    - Supply::getName 으로 Map의 키 타입 명시 → Map<String, Collection<Supply>> 완성
    - `Collectors.counting` : 한 그룹 내 Supply 인스턴스 수 세기 → Map<String, Long> 완성
- **부수효과 없이 간단히 종료 + 가독성 개선하기 위해서는 collect() 연산자를 비롯해 여러 Collectors 의 메서드를 활용하라**

### 8.6. 스트림 내 예외 피하기

```java
class LogBooks {

    static List<LogBook> getAll() throws IOException {
        return Files.walk(Paths.get("/var/log"))
                    .filter(Files::isRegularFile)
                    .filter(LogBook::isLogbook)
                    .map(path -> {
                        try {
                            return new LogBook(path);
                        } catch (IOException e) {
                            throw new UncheckedIOException(e);
                        }
                    })
                    .collect(Collectors.toList());
    }
}
```

- 스트림에서는 검사 예외를 사용할 수 없어, 반드시 스트림 내에서 예외를 처리해야 함
- 따라서 map 연산 내부에서 IOException (검사예외) 를 잡고나서, 비검사예외로 확장되는 UncheckedIOException 으로 변환 한 것
- 자바 함수형 방식에서는 예외, 비검사 예외까지 처리할 적절한 메커니즘이 없음
    - 함수는 예외를 던지거나 잡지 않음

```java
class LogBooks {
    private static Path STORAGE = Paths.get("/var/log");

    static List<LogBook> getAll() throws IOException {
        try (Stream<Path> stream = Files.walk(STORAGE)) {
            return stream.filter(Files::isRegularFile)
                         .filter(LogBook::isLogbook)
                         .flatMap(path -> {
                             try {
                                 return Stream.of(new LogBook(path));
                             } catch (IOException e) {
                                 return Stream.empty();
                             }
                         })
                         .collect(Collectors.toList());
        }
    }
}

class LogBooks2 {
    private static Path STORAGE = Paths.get("/var/log");

    static List<LogBook> getAll() throws IOException {
        try (Stream<Path> stream = Files.walk(STORAGE)) {
            return stream.filter(Files::isRegularFile)
                         .filter(LogBook::isLogbook)
                         .map(path -> {
                             try {
                                 return new LogBook(path);
                             } catch (IOException e) {
                                 return null;
                             }
                         })
                         .filter(Objects::nonNull)
                         .collect(Collectors.toList());
        }
    }
}
```

- try-with-resources 문을 사용하여 자원을 올바르게 닫자
- 검사예외 → 비검사 예외로 변환하지 않음
    - **해당 스트림에서 예외 원소를 간단히 제거하여 문제 발생시 Stream.empty() 를 반환함으로써 예외 발생하더라도 전체 연산 중지 X, 스트림은 입력에 따른 출력만 생성하도록 하자**

### 8.7. 널 대신 옵셔널

- 참조에 어떻게 접근하는지 완벽제어가 가능한 내부 상태라면 null 참조를 사용해도 됨

```java
class Communicator {
    Connection connectionToEarth;

    void establishConnection() {
        // used to set connectionToEarth, but may be unreliable
    }

    Connection getConnectionToEarth() {
        return connectionToEarth;
    }
}

Communicator communicator = new Communicator();
communicator.getConnectionToEarth()
            .send("Houston, we got a problem!");
```

- 여기서 getConnectionToEarth() 가 null 을 반환하면 NPE 가 발생하게 됨

```java
class Communicator {
    Connection connectionToEarth;

    Optional<Connection> getConnectionToEarth() {
        return Optional.ofNullable(connectionToEarth);
    }
}

```

- **connectionToEarth 가 없을수도 있다는 것을 명시하기 위해 `Optional` 을 사용하자**

```java
Communicator communicator = new Communicator();
Connection connection = communicator.getConnectionToEarth()
                                    .orElse(null);
```

- Communcator 내 null 값을 숨기지 않고, Optional 을 통해 접근한 호출자 코드에 null 값 전달
    - 호출자가 부재값을 어떻게 처리할 지 생각해보게 함

```java
Communicator communicationSystem = new Communicator();
communicationSystem.getConnectionToEarth()
                   .ifPresent(connection ->
                        connection.send("Houston, we got a problem!")
                   );

// ifPresent 메서드 명세
public void ifPresent(Consumer<? super T> action) {
        if (value != null) {
            action.accept(value);
        }
    }
```

- Optional 내 값이 null 아닌 경우 전달한 Consumer 를 실행하여 send 함수 호출

### 8.8. 선택 필드나 매개변수 피하기

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

- Optional 에는 부재 (Optional.empty()) or 존재 (presnet) 라는 2가지 상태만 존재함
- 하지만 옵셔널 필드, 메서드 매개변수라면 변수가 null 일수도 있음
→ 따라서 존재, 부재, null 이라는 3가지 상태가 존재하게 됨
→ Optional 이 null 이라는 게 무슨 말인지 이해가 되지 않음

```java
class Communicator {

    Connection connectionToEarth;

    void setConnectionToEarth(Connection connectionToEarth) {
        this.connectionToEarth = Objects.requireNonNull(connectionToEarth);
    }

    Optional<Connection> getConnectionToEarth() {
        return Optional.ofNullable(connectionToEarth);
    }

    void reset() {
        connectionToEarth = null;
    }
}
```

- **필드, 메서드 매개변수 타입에서 Optional 부분을 제거하고, 반환값에만 사용하라**
    - 그래야 Optional.empty() 와 null 간 의미상 혼란을 막을 수 있음
- setter 메서드에서 Optional 타입을 없앴으니까 초기화하기 전에 null 값 검증 필요
- 또한 reset() 을 추가하여, setter 메서드에 명시적으로 null 을 삽입하지 않아도 됨
    - 모든 null 참조는 필드 접근을 완벽히 제어 가능한 클래스 내부에서 발생함

### 8.9. 옵셔널을 스트림으로 사용하기

```java
class BackupJob {

    Communicator communicator;
    Storage storage;

    void backupToEarth() {
        Optional<Connection> connectionOptional =
                communicator.getConnectionToEarth();
        if (!connectionOptional.isPresent()) {
            throw new IllegalStateException();
        }

        Connection connection = connectionOptional.get();
        if (!connection.isFree()) {
            throw new IllegalStateException();
        }

        connection.send(storage.getBackup());
    }
}
```

- Optional 은 0개 또는 1개 원소만 포함하는 특별한 형식의 스트림이므로, 일반 스트림 연산을 바로 적용할 수 있다
- Optional 을 get 으로 꺼내오는 변환작업이 필요한데, 이런 코드를 생략함으로써 가독성 좋은 코드를 만들 수 있다.

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

- **orElseThrow 로 get 으로 꺼내오는 작업들을 없애고 예외를 발생시키자**
