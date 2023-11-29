## 7장. 객체 디자인
### 7.1. 불 매개변수로 메서드 분할

```java
void log(String message, boolean classified) throws IOException {
     if (classified) {
         writeMessage(message, CAPTAIN_LOG);
     } else {
         writeMessage(message, CREW_LOG);
     }
}
```

- 일반적으로 메서드는 하나의 작업에만 특화되어야 하는데, **불 메서드 매개변수는 메서드가 적어도 2가지 작업을 수행함**을 뜻함

```java
logbook.log("Aliens sighted!", true);
logbook.log("Toilet broken.", false);
```

- 호출하는 쪽에서는 boolean 매개변수가 실제 어떤 역할을 하는지 알기 어려워서 코드 이해하기가 어려워짐

```java
void writeToCaptainLog(String message) throws IOException {
   writeMessage(message, CAPTAIN_LOG);
}

void writeToCrewLog(String message) throws IOException {
   writeMessage(message, CREW_LOG);
}
```

- **boolean 메서드 매개변수를 제거하고, 이 매개변수로 구분하던 제어 흐름마다 새 메서드를 추가하라**
    - 새 메서드에 의미있는 이름을 지어서 코드 가독성도 높여라

### 7.2. 옵션 매개변수로 메서드 분할

```java
List<String> readEntries(LocalDate date) throws IOException {
    final List<String> entries = Files.readAllLines(CREW_LOG, StandardCharsets.UTF_8);
    **if (date == null) {
        return entries;
    }**
        
		List<String> result = new LinkedList<>();
    for (String entry : entries) {
        if (entry.startsWith(date.toString())) {
            result.add(entry); 
        }
    }
    return result; 
}
```

```java
List<String> completeLog = logbook.readEntries(null);
List<String> moonLandingLog = logbook.readEntries(LocalDate.of(1969, Month.JULY, 20));
```

- null 을 사용할 수 있다는 것 == 본질적으로 date 매개변수가 선택사항
    - null 매개변수로 메서드 호출 시 **기대하는 바를 가늠하기 어려움**

```java
List<String> **readEntries**(LocalDate date) throws IOException {
   Objects.requireNonNull(date);
        
   List<String> result = new LinkedList<>();
   for (String entry : readAllEntries()) {
        if (entry.startsWith(date.toString())) {
            result.add(entry);
        }
   }
   return result;
}

List<String> **readAllEntries**() throws IOException {
    return Files.readAllLines(CREW_LOG, StandardCharsets.UTF_8);
}
```

```java
List<String> completeLog = logbook.readAllEntries();
List<String> moonLandingLog = logbook.readEntries(moonLanding);
```

- **메서드를 2개로 분할하여 각각 제어 흐름 분기를 표현하라** / readAllEntries, readEntries

### 7.3. 구체 타입보다 추상 타입

```java
class Inventory {
    **LinkedList**<Supply> supplies = new LinkedList();
    void stockUp(**ArrayList**<Supply> delivery) {
        supplies.addAll(delivery);
    }

    **LinkedList**<Supply> getContaminatedSupplies() {
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

- 구체 타입을 사용하게 되면 여러 자료 구조 타입 간 변환이 많은데, 대부분은 실제로는 불필요함

```java
class Inventory {
    **List**<Supply> supplies = new LinkedList();

    void stockUp(**Collection**<Supply> delivery) {
        supplies.addAll(delivery);
    }

    **List**<Supply> getContaminatedSupplies() {
        List<Supply> contaminatedSupplies = new LinkedList<>();
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
inventory.stockUp(delivery);
```

- **구체타입 대신 추상타입을 사용하여, 유연한 코드를 만들어라**
