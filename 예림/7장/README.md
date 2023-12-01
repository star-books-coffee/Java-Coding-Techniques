# 7장. 객체 디자인
## 7.1 불 매개변수로 메서드 분할

### 개선 전 코드

```java
class Logbook {

	static final Path CAPTAIN_LOG = Path.get("/var/captain.log");
	static final Path CREW_LOG =  Path.get("/var/crew.log");
	
	void log(String message, boolean classified) throws IOException {
		if(classified) {
			writeMessage(message, CAPTION_LOG);
		} else {
			writeMessage(message, CREW_LOG);
		}
	}

	void writeMessage(String message, Path location) throws IOException {
		 ... (생략) ...
	}
```

- 불 메서드 매개변수는 메서드가 적어도 두 가지 작업을 수행함을 뜻함
- 위 예제는 log() 메서드의 boolean 매개변수를 사용해 메시지를 공개와 비공개 메시지로 나눔

```java
logbook.log("Aliens sighted!", true);
logbook.log("Toilet broken.", false);
```

- 호출하는 편에서 불 매개변수가 실제로 어떤 역할을 하는지 알기 어려워 코드 이해가 어려워짐
- 또한 기장의 로그(CAPTAIN_LOG) 로직을 변경하면 승무원 로그(CREW_LOG)에도 영향이 갈 수 있음 (두 로그가 서로 연관되어 있음)

### 개선 후 코드

```java
class Logbook {

	static final Path CAPTAIN_LOG = Path.get("/var/captain.log");
	static final Path CREW_LOG =  Path.get("/var/crew.log");
	
	**void writeToCaptainLog(String message) throws IOException** {
		writeMessage(message, CAPTAIN_LOG);
	}
	**void writeToCrewLog(String message) throws IOException** {
		writeMessage(message, CREW_LOG);
	}

	void writeMessage(String message, Path location) throws IOException {
		 ... (생략) ...
	}
```

- 입력 매개변수에 boolean이 쓰인 메서드라면 메서드를 여러 개로 분리함으로써 코드가 향상될 가능성이 높음
- boolean 메서드 매개변수를 제거하고 이 매개변수로 구분하던 각 제어 흐름 경로마다 새 메서드를 추가하는 것임

```java
logbook.**writeToCaptainLog(**"Aliens sighted!");
logbook.**writeToCrewLog(**"Toilet broken. Again...");
```

- 이제 호출하는 코드만 보아도 메서드가 무엇을 하는지 알 수 있음

## 7.2 옵션 매개변수로 메서드 분할

### 개선 전 코드

```java
class Logbook {

    static final Path CREW_LOG = Paths.get("/var/log/crew.log");

    **List<String> readEntries(LocalDate date) throws IOException {**
        final List<String> entries = Files.readAllLines(CREW_LOG,
                StandardCharsets.UTF_8);
        if (date == null) {
            return entries;
        }

        List<String> result = new LinkedList<>();
        for (String entry : entries) {
            if (entry.startsWith(date.toString())) {
                result.add(entry);
            }
        }
        return result;
    }
}
```

- 매개변수로 실제 날짜 값이 아닌 null을 삽입하면 로그 항목 전체를 반환함

```java
List<String> completeLog = logbook.readEntries(null);

final LocalDate moonLanding = LocalDate.of(1969, Month.JULY, 20);
List<String> moonLandingLog = logbook.readEntries(moonLanding);
```

- null을 쓸 수 있다는 것은 본질적으로 date 매개변수가 선택사항이라는 뜻 → boolean 매개변수를 다른 형태로 바꾼 것
- 옵션 매개변수를 포함하는 메서드도 두 가지 이상의 작업을 수행하는데, null 매개변수로 메서드를 호출했을 때 기대하는 바를 가늠하기 어려움

### 개선 후 코드

```java
class Logbook {

    static final Path CREW_LOG = Paths.get("/var/log/crew.log");

    **List<String> readEntries(LocalDate date) throws IOException {**
        Objects.requireNonNull(date);
        
        List<String> result = new LinkedList<>();
        for (String entry : **readAllEntries()**) {
            if (entry.startsWith(date.toString())) {
                result.add(entry);
            }
        }
        return result;
    }

    **List<String> readAllEntries() throws IOException {**
        return Files.readAllLines(CREW_LOG, StandardCharsets.UTF_8);
    }
}
```

- 7.1과 같은 해결방법으로, 메서드 두 개로 분할해 각각 제어 흐름 분기를 하나씩 표현함

```java
List<String> completeLog = logbook.**readAllEntries();**

final LocalDate moonLanding = LocalDate.of(1969, Month.JULY, 20);
List<String> moonLandingLog = logbook.**readEntries(moonLanding);**
```

- 메서드명을 통해 모든 항목을 읽겠다고 명확히 전달하니 훨씬 이해가 쉬움

## 7.3 구체 타입보다 추상 타입

- 인터페이스와 클래스는 변수에 더 광범위한 타입 계층 구조를 형성함. 변수에 더 추상적인 타입을 사용할수록 코드가 더 유연해짐

```java
class Inventory {
    LinkedList<Supply> supplies = new LinkedList();

    void stockUp(ArrayList<Supply> delivery) {
        supplies.addAll(delivery);
    }

    LinkedList<Supply> getContaminatedSupplies() {
        LinkedList<Supply> contaminatedSupplies = new LinkedList<>();
        for (Supply supply : supplies) {
            if (supply.isContaminated()) {
                contaminatedSupplies.add(supply);
            }
        }
        return contaminatedSupplies;
    }
}
```

```java
Stack<Supply> delivery = cargoShip.unload();
ArrayList<Supply> loadableDelivery = new ArrayList<>(delivery);
inventory.stockUp(loadableDelivery);
```

- Stack을 사용해 제품을 후입 선출(LIFO) 순으로 전달
- 여러 자료 구조 타입 간 변환이 많은데 그 중 대부분은 실제로 불필요함 (ArrayList 생성 → LinkedList로 변환)
- Inventory를 변경할 경우 다른 부분에 쉽게 영향을 미침

### 개선 후 코드

```java
class Inventory {
    List<Supply> supplies = new LinkedList();

    void stockUp(**Collection<Supply> delivery**) {
        supplies.addAll(delivery);
    }

    **List<Supply>** getContaminatedSupplies() {
        **List<Supply> contaminatedSupplies** = new LinkedList<>();
        for (Supply supply : supplies) {
            if (supply.isContaminated()) {
                contaminatedSupplies.add(supply);
            }
        }
        return contaminatedSupplies;
    }
}
```

- **추상 타입**을 사용하여 문제 해결
- 달라진 점은 다음과 같다
    1. supplies 필드에 LinkedList 대신 List 인터페이스 타입을 사용함
        
        ⇒ 제품은 순서대로 저장되지만 어떻게 저장되는지는 알 수 없음
        
    2. stockUp() 메서드가 어떤 Collection이든 허용함
        
        ⇒ Collection은 자바에서 자료 구조에 객체를 저장하는 가장 기본적인 인터페이스이므로 자바의 어떤 복잡한 자료 구조이든 이 메서드로 전달 가능
        
    3. getContaminatedSupplies() 메서드가 더 구체적인 타입이 아닌 List를 반환함
        
        ⇒ 제품은 반드시 정렬된 상태로 반환되지만 내부적으로 리스트를 어떻게 구현했는지는 알려지지 않아 코드가 더 유연해짐
        

```java
Stack<Supply> delivery = cargoShip.unload();
inventory.stockUp(delivery);
```

- 이제 Stack이 Collection이라 Inventory는 아무 변환 없이 바로 Stack을 받아들임
- 심지어 Inventory는 Set, List, 필요하다면 Vector와 그 외 특수한 목적으로 만들어진 자료 구조까지 로드 가능


## 7.2 옵션 매개변수로 메서드 분할

### 개선 전 코드

```java
class Logbook {

    static final Path CREW_LOG = Paths.get("/var/log/crew.log");

    **List<String> readEntries(LocalDate date) throws IOException {**
        final List<String> entries = Files.readAllLines(CREW_LOG,
                StandardCharsets.UTF_8);
        if (date == null) {
            return entries;
        }

        List<String> result = new LinkedList<>();
        for (String entry : entries) {
            if (entry.startsWith(date.toString())) {
                result.add(entry);
            }
        }
        return result;
    }
}
```

- 매개변수로 실제 날짜 값이 아닌 null을 삽입하면 로그 항목 전체를 반환함

```java
List<String> completeLog = logbook.readEntries(null);

final LocalDate moonLanding = LocalDate.of(1969, Month.JULY, 20);
List<String> moonLandingLog = logbook.readEntries(moonLanding);
```

- null을 쓸 수 있다는 것은 본질적으로 date 매개변수가 선택사항이라는 뜻 → boolean 매개변수를 다른 형태로 바꾼 것
- 옵션 매개변수를 포함하는 메서드도 두 가지 이상의 작업을 수행하는데, null 매개변수로 메서드를 호출했을 때 기대하는 바를 가늠하기 어려움

### 개선 후 코드

```java
class Logbook {

    static final Path CREW_LOG = Paths.get("/var/log/crew.log");

    **List<String> readEntries(LocalDate date) throws IOException {**
        Objects.requireNonNull(date);
        
        List<String> result = new LinkedList<>();
        for (String entry : **readAllEntries()**) {
            if (entry.startsWith(date.toString())) {
                result.add(entry);
            }
        }
        return result;
    }

    **List<String> readAllEntries() throws IOException {**
        return Files.readAllLines(CREW_LOG, StandardCharsets.UTF_8);
    }
}
```

- 7.1과 같은 해결방법으로, 메서드 두 개로 분할해 각각 제어 흐름 분기를 하나씩 표현함

```java
List<String> completeLog = logbook.**readAllEntries();**

final LocalDate moonLanding = LocalDate.of(1969, Month.JULY, 20);
List<String> moonLandingLog = logbook.**readEntries(moonLanding);**
```

- 메서드명을 통해 모든 항목을 읽겠다고 명확히 전달하니 훨씬 이해가 쉬움

## 7.3 구체 타입보다 추상 타입

- 인터페이스와 클래스는 변수에 더 광범위한 타입 계층 구조를 형성함. 변수에 더 추상적인 타입을 사용할수록 코드가 더 유연해짐

```java
class Inventory {
    LinkedList<Supply> supplies = new LinkedList();

    void stockUp(ArrayList<Supply> delivery) {
        supplies.addAll(delivery);
    }

    LinkedList<Supply> getContaminatedSupplies() {
        LinkedList<Supply> contaminatedSupplies = new LinkedList<>();
        for (Supply supply : supplies) {
            if (supply.isContaminated()) {
                contaminatedSupplies.add(supply);
            }
        }
        return contaminatedSupplies;
    }
}
```

```java
Stack<Supply> delivery = cargoShip.unload();
ArrayList<Supply> loadableDelivery = new ArrayList<>(delivery);
inventory.stockUp(loadableDelivery);
```

- Stack을 사용해 제품을 후입 선출(LIFO) 순으로 전달
- 여러 자료 구조 타입 간 변환이 많은데 그 중 대부분은 실제로 불필요함 (ArrayList 생성 → LinkedList로 변환)
- Inventory를 변경할 경우 다른 부분에 쉽게 영향을 미침

### 개선 후 코드

```java
class Inventory {
    List<Supply> supplies = new LinkedList();

    void stockUp(**Collection<Supply> delivery**) {
        supplies.addAll(delivery);
    }

    **List<Supply>** getContaminatedSupplies() {
        **List<Supply> contaminatedSupplies** = new LinkedList<>();
        for (Supply supply : supplies) {
            if (supply.isContaminated()) {
                contaminatedSupplies.add(supply);
            }
        }
        return contaminatedSupplies;
    }
}
```

- **추상 타입**을 사용하여 문제 해결
- 달라진 점은 다음과 같다
    1. supplies 필드에 LinkedList 대신 List 인터페이스 타입을 사용함
        
        ⇒ 제품은 순서대로 저장되지만 어떻게 저장되는지는 알 수 없음
        
    2. stockUp() 메서드가 어떤 Collection이든 허용함
        
        ⇒ Collection은 자바에서 자료 구조에 객체를 저장하는 가장 기본적인 인터페이스이므로 자바의 어떤 복잡한 자료 구조이든 이 메서드로 전달 가능
        
    3. getContaminatedSupplies() 메서드가 더 구체적인 타입이 아닌 List를 반환함
        
        ⇒ 제품은 반드시 정렬된 상태로 반환되지만 내부적으로 리스트를 어떻게 구현했는지는 알려지지 않아 코드가 더 유연해짐
        

```java
Stack<Supply> delivery = cargoShip.unload();
inventory.stockUp(delivery);
```

- 이제 Stack이 Collection이라 Inventory는 아무 변환 없이 바로 Stack을 받아들임
- 심지어 Inventory는 Set, List, 필요하다면 Vector와 그 외 특수한 목적으로 만들어진 자료 구조까지 로드 가능

## 7.4 가변 상태보다 불변 상태 사용하기

### 개선 전 코드

```java
class Distance {
    **DistanceUnit unit;
    double value;**

    Distance(DistanceUnit unit, double value) {
        this.unit = unit;
        this.value = value;
    }

    static Distance km(double value) {
        return new Distance(DistanceUnit.KILOMETERS, value);
    }

    void add(Distance distance) {
        distance.convertTo(unit);
        **value += distance.value;**
    }

    void convertTo(DistanceUnit otherUnit) {
        double conversionRate = unit.getConversionRate(otherUnit);
        **unit = otherUnit;**
        **value = conversionRate * value;**
    }
}
```

- 기본적으로 객체의 상태는 불변이고, 가능하면 객체를 불변으로 만들어야 잘못 사용할 경우가 적음

```java
Distance toMars = new Distance(DistanceUnit.KILOMETERS, 56_000_000);
Distance marsToVenus = new Distance(DistanceUnit.LIGHTYEARS, 0.000012656528);
Distance toVenusViaMars = toMars;
toVenusViaMars.add(marsToVenus);
```

- toVenusViaMars와 toMars가 가리키는 객체가 같음 → toVenusViaMars.add(marsToVenus)를 호출하면 toMars 값까지 간접적으로 변환하게 됨
- 이 문제를 컴파일러로 미연에 방지할 수 있음

### 개선 후 코드

```java
final class Distance {
    **final DistanceUnit unit;
    final double value;**

    Distance(DistanceUnit unit, double value) {
        this.unit = unit;
        this.value = value;
    }

    Distance add(Distance distance) {
        **return new Distance(unit, value + distance.convertTo(unit).value);**
    }

    Distance convertTo(DistanceUnit otherUnit) {
        double conversionRate = unit.getConversionRate(otherUnit);
        **return new Distance(otherUnit, conversionRate * value);**
    }
}
```

- 객체는 유효하지 않은 변경이 일어나지 않도록 스스로 보호해야 하는데 가변성을 제한하면 가능함
- 생성자의 value와 unit 필드에 final 키워드를 설정했기 때문에 이후로는 바꿀 수 없고, 거리를 계산하려면 매번 새로운 인스턴스가 필요함

```java
Distance toMars = new Distance(DistanceUnit.KILOMETERS, 56_000_000);
Distance marsToVenus = new Distance(DistanceUnit.LIGHTYEARS, 0.000012656528);
Distance toVenusViaMars = toMars.add(marsToVenus)
                              .convertTo(DistanceUnit.MILES);
```

- 객체를 더 많이 생성한다는 단점은 있지만 자바에서 작은 객체는 적은 비용이 듦

## 7.5 상태와 동작 결합하기

### 개선 전 코드

```java
class Hull {
    int holes;
}

class HullRepairUnit {

    void repairHole(Hull hull) {
        if (isIntact(hull)) {
            return;
        }
        hull.holes--;
    }

    boolean isIntact(Hull hull) {
        return hull.holes == 0;
    }
}
```

- 동작만 있고 상태가 없는 클래스를 만들면 정보 은닉이 불가능해지고 코드가 더 장황해짐
- Hull 객체에 접근하고 수정하는 것을 막기 어려움
- hull 매개변수 검증도 하지 않음
- 이러한 경우에 해결하는 방법은 다음과 같음
    - 너무 큰 클래스나 자신에게 속한 메서드 매개변수만 연산하는 클래스를 찾음
    - 비슷한 작업을 수행하는 변수와 메서드를 하나의 클래스로 묶어 클래스를 간소화함
    - 전후 비교를 수행해 디자인이 정말 좋아졌는지 확인

### 개선 후 코드

```java
class Hull {
    int holes;

    void repairHole() {
        if (isIntact()) {
            return;
        }
        holes--;
    }

    boolean isIntact() {
        return holes == 0;
    }
}
```

- Hull 클래스 스스로 기능을 제공해 상태와 기능을 제공함
- 원래는 매개변수를 검증해야하는데 하지 않아도 되고, 게터와 세터를 두어 holes 속성을 외부로 노출할 필요도 없음
- **메서드 내에서 입력 매개변수만 다루고 자신이 속한 클래스의 인스턴스 변수는 다르지 않는 경우를 유심히 살펴보아야 함**
    - 이것은 상태와 동작이 분리되었다는 의미이고, 이러한 메서드로는 정보 은닉이 불가능함

## 7.6 참조 누수 피하기

### 개선 전 코드

```java
class Inventory {

    private final List<Supply> supplies;

    Inventory(List<Supply> supplies) {
        this.supplies = supplies;
    }

    List<Supply> getSupplies() {
        return supplies;
    }
}
```

- 명백하지 않은 객체에는 외부에서 접근할 수 있는 내부 상태가 항상 있음
- 이러한 상태를 어떤 방식으로 조작할지 신중히 결정해야 심각한 버그를 막을 수 있음

```java
List<Supply> externalSupplies = new ArrayList<>();
Inventory inventory = new Inventory(externalSupplies);

inventory.getSupplies().size(); // == 0
externalSupplies.add(new Supply("Apple"));
inventory.getSupplies().size(); // == 1

inventory.getSupplies().add(new Supply("Banana"));
inventory.getSupplies().size(); // == 2
```

- inventory는 내부의 제품 리스트를 전혀 보호하지 않음
    - externalSupplies 리스트에 지품을 추가하거나 getSupplies()가 반환한 리스트에 변경 연산을 수행하면 재고 상태가 바뀜
- 나중에 예외를 발생시킬 수 있는 null마저 생성자에 전달할 수 있음
- 원인은 메모리에 들어있는 리스트가 new ArrayList<>()로 생성한 리스트 하나 뿐이기 때문임
    - 사실상 Inventory는 내부 구조로의 참조를 게터를 통해 바깥으로 노출하는 셈임

### 개선 후 코드

```java
class Inventory {

    private final List<Supply> supplies;

    Inventory(List<Supply> supplies) {
        **this.supplies = new ArrayList<>(supplies);**
    }

    List<Supply> getSupplies() {
        **return Collections.unmodifiableList(supplies);**
    }
}
```

- 전달한 리스트의 참조가 아니라 리스트 내 Supply 객체로 ArrayList를 채움
- unmodifiableList()로 래핑한 후 노출하여 읽기 접근만 가능하므로 리스트에 원소를 추가하려면 이러한 기능을 하는 명시적인 메서드를 작성해야 함
- 내부 ArrayList 인스턴스는 Inventory 내에서 프라이빗이므로 인스턴스로의 참조가 클래스 밖으로 나가지 않음

```java
List<Supply> externalSupplies = new ArrayList<>();
Inventory inventory = new Inventory(externalSupplies);

inventory.getSupplies().size(); // == 0
externalSupplies.add(new Supply("Apple"));
**inventory.getSupplies().size(); // == 0**

// UnsupportedOperationException
**inventory.getSupplies().add(new Supply("Banana"));**
```

- inventory의 내부 상태에 전혀 영향이 없고, getSupplies()가 반환한 리스트를 수정하려고 하면 UnsupportedOperationException 발생
- 이러한 기법을 **방어복사(defensive copying)**이라고도 부름
- 게터와 세터 둘 다 보호해야 하고, 애당초 세터를 허용하지 않는 편이 훨씬 편함

## 7.7 널 반환하지 않기

### 개선 전 코드

```java
class SpaceNations {

    static List<SpaceNation> nations = Arrays.asList(
            new SpaceNation("US", "United States"),
            new SpaceNation("RU", "Russia")
    );

    static SpaceNation getByCode(String code) {
        for (SpaceNation nation : nations) {
            if (nation.getCode().equals(code)) {
                return nation;
            }
        }
        **return null;**
    }
}
```

- 메서드 호출 시 적절한 반환값이 없다고 null을 반환하면 안됨. 프로그램의 안정성을 해치는 행위임

```java
String us = SpaceNations.getByCode("US").getName();
// -> "United States"
**String anguilla = SpaceNations.getByCode("AI").getName();
// -> NullPointerException**
```

- NullPointException이 쉽게 발생함
- 메서드가 null을 반환할 가능성이 있으니 매번 명시적으로 반환값을 확인해야 함

### 개선 후 코드

```java
class SpaceNations {

    **/** Null object. */
    static final SpaceNation UNKNOWN_NATION = new SpaceNation("", "");**

    static List<SpaceNation> nations = Arrays.asList(
            new SpaceNation("US", "United States"),
            new SpaceNation("RU", "Russia")
    );

    static SpaceNation getByCode(String code) {
        for (SpaceNation nation : nations) {
            if (nation.getCode().equals(code)) {
                return nation;
            }
        }
        **return UNKNOWN_NATION;**
    }
}
```

- `IllegalArgumentException`이나 `NoSuchElementException`과 같은 예외를 던져 호출하는 쪽에서 명시적으로 문제를 처리하도록 하는 방법도 있음
- 그러나 위 예제에서는 **널 객체 패턴**을 권함.
    - 널 객체, 즉 객체에 실질적인 값이 없음을 명시적으로 표현한 객체를 반환하는 방식임

```java
String us = SpaceNations.getByCode("US").getName(); // -> "United States"
**String anguilla = SpaceNations.getByCode("AI").getName(); // -> ""**
```

- UNKNONW_NATION이 나올 경우 어떻게 대응할지는 여전히 호출하는 쪽에 달려있지만, **값을 무시하든 예외를 던지든 선택의 여지가 생김**
- 널 객체는 빈 문자열, 빈 컬렉션, 또는 예제에서처럼 특수 클래스 인스턴스 등 다양한 형태로 표현됨. 어떤 형태를 띠든 공통 목표는 **“비용이 막대한 실수”가 일어나지 않게 하는 것**
