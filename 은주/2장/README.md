## 2장. 코드 스타일 레벨 업

### 2.1. 매직 넘버를 상수로 대체

- `매직 넘버` : 표면상 의미 없는 숫자지만 프로그램의 동작을 제어함

```java
class CruiseControl {

    private double targetSpeedKmh;

    void setPreset(int speedPreset) {
        if (speedPreset == 2) {
            setTargetSpeedKmh(16944);
        } else if (speedPreset == 1) {
            setTargetSpeedKmh(7667);
        } else if (speedPreset == 0) {
            setTargetSpeedKmh(0);
        }
    }

    void setTargetSpeedKmh(double speed) {
        targetSpeedKmh = speed;
    }
}
```

- **static final 로 상수를 선언하고, 숫자값을 상수로 변경하여 코드를 훨씬 명확하게 만들어라**

```java
class CruiseControl {

    static final int STOP_PRESET = 0;
    static final int PLANETARY_SPEED_PRESET = 1;
    static final int CRUISE_SPEED_PRESET = 2;

    static final double CRUISE_SPEED_KMH = 16944;
    static final double PLANETARY_SPEED_KMH = 7667;
    static final double STOP_SPEED_KMH = 0;

		private double targetSpeedKmh;

    void setPreset(int speedPreset) {
        if (speedPreset == CRUISE_SPEED_PRESET) {
            setTargetSpeedKmh(CRUISE_SPEED_KMH);
        } else if (speedPreset == PLANETARY_SPEED_PRESET) {
            setTargetSpeedKmh(PLANETARY_SPEED_KMH);
        } else if (speedPreset == STOP_PRESET) {
            setTargetSpeedKmh(STOP_SPEED_KMH);
        }
    }

    void setTargetSpeedKmh(double speed) {
        targetSpeedKmh = speed;
    }
}
```

### 2.2. 정수 상수 대신 열거형

- 위의 코드는 입력 매개변수인 speedPreset 에 어떤 값이라도 넣을 수 있고, STOP_PRESET 과 같은 상수를 사용해야 한다고 강제하지 않음
- 컴파일러가 유효하지 않은 값을 거절하게 하기 위해, **가능한 옵션을 모두 열거할 수 있다면 항상 정수 대신 enum 타입을 사용하라**
    - 아래 코드를 통해, 존재하지 않는 speedPreset 은 더이상 메서드의 매개변수로 전달할 수 없고, 시도하더라도 **컴파일러에 의해 중지**됨

```java
class CruiseControl {
    private double targetSpeedKmh;

    void setPreset(SpeedPreset speedPreset) {
        Objects.requireNonNull(speedPreset);

        setTargetSpeedKmh(speedPreset.speedKmh);
    }

    void setTargetSpeedKmh(double speedKmh) {
        targetSpeedKmh = speedKmh;
    }
}
enum SpeedPreset {
    STOP(0), PLANETARY_SPEED(7667), CRUISE_SPEED(16944);

    final double speedKmh;

    SpeedPreset(double speedKmh) {
        this.speedKmh = speedKmh;
    }
}
```

### 2.3. For 루프 대신 For-Each

- **인덱스 변수를 실수할 여지가 있음**
    - protected 가 아니여서 언제든지 덮어쓸 수 있음
    - 종료기준 (<, ≤) 에 따라 IndexOutOfBoundsExceptions 발생 가능성 존재
- For-Each 를 사용할 경우, 배열과 Set 처럼 인덱싱이 불가능한 컬렉션에서도 동작함
- 인덱스로 순회하는 방식은 거의 없으며, 드물게 컬렉션의 특정 부분만 순회할 경우 사용

### 2.4. 순회하며 컬렉션 수정하지 않기

```java
class Inventory {
    private List<Supply> supplies = new ArrayList<>();
    void disposeContaminatedSupplies() {
        for (Supply supply : supplies) {
            if (supply.isContaminated()) {
                **supplies.remove(supply);**
            }
        }
    }
}
```

- List 인터페이스 표준 구현이나 Set, Queue 와 같은 Collection 인터페이스 구현은 `ConcurrentModificationException` 을 던짐 (Collection 을 순회하는 동안 그 컬렉션을 수정하지 못한다는 뜻)
    - 자바의 컴파일 타임 검사로는 이 오류를 잡아내지 못함

```java
class Inventory {
    private List<Supply> supplies = new ArrayList<>();
    void disposeContaminatedSupplies() {
        Iterator<Supply> iterator = supplies.iterator();
        while (iterator.hasNext()) {
            if (iterator.next().isContaminated()) {
                iterator.remove();
            }
        }
    }
}
```

- `ConcurrentModificationException` 를 일으키지 않으면서 올바르게 수행하는 방법
    - 리스트를 순회하며, 변질된 아이템을 찾고 그 후에 앞에서 발견한 제품을 모두 제거하는 것 (먼저 순회하고 나중에 수정하는 접근법)
    - iterator 는 순회 중에도 모든 작업을 올바르게 수행함 (for-each 루프도 iterator 에 기반함)
- CopyOnWriteArrayList 와 같은 특수 List 구현은 순회하며 수정하기도 함
- 리스트에 원소 추가/제거할 때마다 매번 전체 리스트를 복사하고 싶다면, Collection.removeIf() 메소드를 사용할 수 있음

### 2.5. 순회하며 계산 집약적 연산하지 않기

```java
class Inventory {
    private List<Supply> supplies = new ArrayList<>();
    List<Supply> find(String regex) {
        List<Supply> result = new LinkedList<>();
        for (Supply supply : supplies) {
            if (**Pattern.matches**(regex, supply.toString())) {
                result.add(supply);
            }
        }
        return result;
    }
}
```

- String 표현식인 regex 를 가져와 regex 로부터 특수한 목적의 `오토마톤` 을 만듦
    - 오토마톤 : 패턴을 따르는 문자열만 허용하고 나머지는 모두 거절함
- **Pattern.matches**(regex, supply.toString()) 는 오토마톤을 컴파일하여 매칭 연산 수행
    - 정규식 오토마톤 컴파일은 클래스 컴파일처럼 시간, 처리 전력 소모
    - 반복할 때마다 정규식 컴파일은 성능 저하를 야기함

```java
class Inventory {
    private List<Supply> supplies = new ArrayList<>();
    List<Supply> find(String regex) {
        List<Supply> result = new LinkedList<>();
        **Pattern pattern = Pattern.compile(regex);**
        for (Supply supply : supplies) {
            if (**pattern.matcher**(supply.toString()).matches()) {
                result.add(supply);
            }
        }
        return result;
    }
}
```

- 정규식 컴파일은 한번만 수행하고, 반복 문 내에서는 **pattern.matcher**(supply.toString()) 로 검색하는 작업을 수행함

### 2.6. 새 줄로 그루핑

```java
enum DistanceUnit {

    MILES, KILOMETERS;

    static final double MILE_IN_KILOMETERS = 1.60934;
    static final int IDENTITY = 1;
    static final double KILOMETER_IN_MILES = 1 / MILE_IN_KILOMETERS;

    double getConversionRate(DistanceUnit unit) {
        if (this == unit) {
            return IDENTITY;
        }
        if (this == MILES && unit == KILOMETERS) {
            return MILE_IN_KILOMETERS;
        } else {
            return KILOMETER_IN_MILES;
        }
    }
}
```

```java
enum DistanceUnit {

    MILES, KILOMETERS;

    static final int IDENTITY = 1;

    static final double MILE_IN_KILOMETERS = 1.60934;
    static final double KILOMETER_IN_MILES = 1 / MILE_IN_KILOMETERS;

    double getConversionRate(DistanceUnit unit) {
        if (this == unit) {
            return IDENTITY;
        }

        if (this == MILES && unit == KILOMETERS) {
            return MILE_IN_KILOMETERS;
        } else {
            return KILOMETER_IN_MILES;
        }
    }
}
```

- 필드와 메소드 내의 if 문 사이를 **새 줄로 분리**하여 코드 이해도를 향상시켜라
- 연관된 코드, 개념은 함께 그루핑하고 **서로 다른 그룹은 빈 줄로 각각 분리**해야 한다

### 2.7. 이어붙이기 대신 서식화

```java
class Mission {
	Logbook logbook;
	LocalDate start;

	void update(String author, String message) {
		LocalDate today = LocalDate.now();
		String month = String.valueOf(today.getMonthValue());
		String formattedMonth = month.length() < 2 ? "0" + month : month;
		String entry = author.toUpperCase() + ": [" + formattedMonth + "-" + today.getDayOfMonth() + "-" + today.getYear() + "](Day " + (ChronoUnit.Days.between(start, today) + 1) + ")> " + message + System.lineSperator();
		logbook.write(entry);
	}
}
```

- 문제 : 출력이 실제로 어떤 모습일지 알기 어려움
- String, Int 의 + 연산이 마구 섞여 있음

```java
...(생략)...

void update(String author, String message) {
		LocalDate today = LocalDate.now();
		String entry = String.format("%S: [%tm-%<te-%<tY](Day %d)> %s%n", author, today, ChronoUnit.DAYS.between(start, today) + 1, message);
		logbook.write(entry);
}
```

- String 레이아웃 (String 을 **어떻게 출력할지**) 와 데이터 (**무엇을 출력할지**) 를 분리하라
- String.format(), System.out.printf() 와 같은 포맷 메소드를 사용하라
- “%S: [%tm-%<te-%<tY](Day %d)> %s%n" 이 최종적으로 무엇을 출력할지는 알기 어렵지만, 이 방식은 문서화가 잘 된 표준이자 어수선한 코드를 해결할 훌륭한 대안이다
- 문자열이 길면 강력한 템플릿 엔진인 [StringTemplate](https://www.stringtemplate.org/) 을 사용하라

### 2.8. 직접 만들지 말고 자바 API 사용하기

```java
class Inventory {
    private List<Supply> supplies = new ArrayList<>();

    int getQuantity(Supply supply) {
        if (supply == null) {
            throw new NullPointerException("supply must not be null");
        }

        int quantity = 0;
        for (Supply supplyInStock : supplies) {
            if (supply.equals(supplyInStock)) {
                quantity++;
            }
        }

        return quantity;

    }
}
```

- API 에 있는 기능을 다시 구현하지 말고 가능하면 재사용하라

```java
class Inventory {
    private List<Supply> supplies = new ArrayList<>();

    int getQuantity(Supply supply) {
        Objects.requireNonNull(supply, "supply must not be null");

        return Collections.frequency(supplies, supply);
    }
}
```

- Objects 유틸리티 클래스의 requireNonNull() 메서드를 사용하여 객체가 null 일 경우 NPE 던짐
- Collections내 객체 출현 횟수를 세는 frequency 메서드를 사용
- 직접 작성한 코드는 API 보다 버그를 일으킬 가능성이 크므로, API 를 적극 활용하라
