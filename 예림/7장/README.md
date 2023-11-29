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
