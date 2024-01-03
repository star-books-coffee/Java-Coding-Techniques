# 2장. 코드 스타일 레벨업
## 2.1 매직 넘버를 상수로 대체

- 매직 넘버 : 어떤 코드나 프로그램에서 하드코딩된 숫자를 가리키는 용어
- **매직 넘버**가 있으면 코드를 이해하기 어려워지고 오류 발생이 쉬움

### 개선 전 코드

```java
class CruiseControl {
	
	private double targetSpeedkmh;

	void setPreset(int speedPreset) {
		if(speedPreset == 2) {
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

- 메서드 내부에 대한 정교한 지식이 필요하고 잘못 사용되기 쉬움

### 개선 후 코드

```java
class CruiseControl {

	static final int STOP_PRESET = 0;
	static final int PLANETARY_SPEED_PRESET = 1;
	static final int CRUISE_SPEED_PRESET = 2;

	static final double CRUISE_SPEED_KMH = 16944;
	static final double PLANTARY_SPEED_KMH = 7667;
	static final double STOP_SPEED_KHM = 0;
	
	private double targetSpeedkmh;

	void setPreset(int speedPreset) {
		if(speedPreset == CRUISE_SPEED_PRESET) {
			setTargetSpeedKmh(CRUISE_SPEED_KMH);
		} else if (speedPreset == PLANETARY_SPEED_PRESET) {
			setTargetSpeedKmh(PLANTARY_SPEED_KMH);
		} else if (speedPreset == STOP_PRESET) {
			setTargetSpeedKmh(STOP_SPEED_KHM);
		}
	}

	void setTargetSpeedKmh(double speed) {
		targetSpeedKmh = speed;
	}
}
```

- 상수 사용

## 2.2 정수 상수 대신 열거형

- 매직 넘버보다 상수가 훨씬 낫지만, 옵션을 모두 열거할 수 있다면 자바 타입 시스템이 제공하는 방법이 더 나음
- 유효하지 않은 정숫값을 setPreset()에 넣어도 메서드가 특별히 하는 일이 없음
- 자바와 같은 정적 타입 언어는 이러한 오류를 프로그램 실행 전 발견할 수 있는 기능 제공 → enum

### 개선 후 코드

```java
class CruiseControl {
	
	private double targetSpeedkmh;

	void setPreset(int speedPreset) {
		**Objects.requiredNotNull(speedPreset);**
		
		setTargetSpeedKmh(speedPreset.speedKmh);
	}

	void setTargetSpeed(double speedkmh) {
		targetSpeedKmh = speedkmh;
	}
}

enum SpeedPreset {
	STOP(0), PLANETARY_SPEED(16944), CRUISE_SPEED(16944);

	final double speedKmh;

	SpeedPreset(double speedKmh) {
		this.speedKmh = speedKmh;
	}
}
```

- 예제처럼 가능한 옵션을 모두 열거할 수 있다면 항상 정수 대신 enum 타입을 사용하라
- 주요 장점은 존재하지 않는 SpeedPreset을 더이상 SetPreset() 메서드로 넣을 수 없음 (시도해도 자바 컴파일러가 중지시킴)

## 2.3 For 루프 대신 For-Each

### 개선 전 코드

```java
class LaunchChecklist {
	
	List<String> checks = Arrays.asList("Cabin Pressure",
					"Communication",
					"Engine");
	
	Status prepareForTakeoff(Commander commander) {
		for(int i = 0; i < check.size(); i++) {
			boolean shouldAbortTakeoff = commaner.isFailing(checks.get(i));
			if(shouldAbortTakeoff) {
				return Status.ABORT_TAKE_OFF;
			}
		}
		return Status.READY_FOR_TAKE_OFF;
	}
}
```

- 리스트 내 다음 원소에 접근할 때가 아니면 인덱스를 쓰지 않으므로 인덱스를 계속 추적할 필요가 없음
- 인덱스 변수에는 실수할 여지가 있음
- 인덱스 변수가 제공하는 정보를 자세히 알아야 할 경우는 드물고, 이럴 때에는 세부 순회 내용은 보호할 수 없지만 적어도 프로그래머에게 숨기는 식으로 작성해야 함

### 개선 후 코드

```java
class LaunchChecklist {
	
	...(생략)...
	
	Status prepareForTakeoff(Commander commander) {
		for(String check : checks) {
			boolean shouldAbortTakeoff = commander.isFailing(check);
			if(shouldAbortTakeoff) {
				return Status.ABORT_TAKE_OFF;
			}
		}
		return Status.READY_FOR_TAKE_OFF;
	}
}
```

- 반복 인덱스를 더 이상 다루지 않아도 됨
- 인덱싱되지 않은 컬렉션에도 동작

## 2.4 순회하며 컬렉션 수정하지 않기

### 개선 전 코드

```java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    void disposeContaminatedSupplies() {
        for (Supply supply : supplies) {
            if (supply.isContaminated()) {
                supplies.remove(supply);
            }
        }
    }
}
```

- 자료 구조를 바꾸려면 조심해야 함. 프로그램이 충돌할 위험이 있음
- 위 예제에서는 재고 목록 내 한 supply라도 오염되었을 경우 충돌함
- 직관적인 방법은 리스트를 순회하며 변질된 제품을 찾고 **그 후** 앞에서 발견한 제품을 모두 제거하는 것이지만, 시간과 메모리가 더 듦

### 개선 후 코드

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

- supplies 컬렉션의 Iterator를 활용

## 2.5 순회하며 계산 집약적 연산하지 않기

### 개선 전 코드

```java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    List<Supply> find(String regex) {
        List<Supply> result = new LinkedList<>();
        for (Supply supply : supplies) {
            if (Pattern.matches(regex, supply.toString())) {
                result.add(supply);
            }
        }
        return result;
    }
}
```

- 자바 API의 java.util.regex.Pattern 클래스는 정규식을 만들고 실행하는 다양한 메서드 제공
- matches() 함수는 정규식인 String과 검색할 String 제공하면 String 표현식인 regex를 가져와 regex로부터 특수한 목적의 오토마톤 생성
- 이 오토마톤은 패턴을 따르는 문자열만 허용하고 나머지는 모두 거절
- 정규식 오토마톤 컴파일은 시간과 처리 전력 소모
- 보통 일회성 동작이지만 위 예제에서는 반복할 때마다 정규식을 컴파일

### 개선 후 코드

```java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    List<Supply> find(String regex) {
        List<Supply> result = new LinkedList<>();
        **Pattern pattern = Pattern.compile(regex);**
        for (Supply supply : supplies) {
            if (pattern.matcher(supply.toString()).matches()) {
                result.add(supply);
            }
        }
        return result;
    }
}
```

- 메서드를 호출할 때 정규식을 딱 한 번만 컴파일
- Pattern.matches() 호출에 들어 있는 두 연산을 분해

⇒ 정규식을 조금만 고쳐서 사용하면 성능을 크게 높일 수 있음

## 2.6 새 줄로 그루핑

### 개선 전 코드

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

- 별개 블록을 새 줄로 분리하면 코드 이해도를 향상 시킬 수 있음

### 개선 후 코드

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

- IDENTITY 필드를 다른 상수와 분리
- 두 if 블록을 서로 분리함 (오호)
- 연관된 코드와 개념은 함께 그루핑하고 서로 다른 그룹은 빈 줄로 각각 분리해야 함

## 2.7 이어붙이기 대신 서식화

### 개선 전 코드

```java
```java
class Mission {

    Logbook logbook;
    LocalDate start;

    void update(String author, String message) {
        LocalDate today = LocalDate.now();
        String month = String.valueOf(today.getMonthValue());
        String formattedMonth = month.length() < 2 ? "0" + month : month;
        String entry = author.toUpperCase() + ": [" + formattedMonth + "-" +
                today.getDayOfMonth() + "-" + today.getYear() + "](Day " +
                (ChronoUnit.DAYS.between(start, today) + 1) + ")> " +
                message + System.lineSeparator();
        logbook.write(entry);
    }
}
```

- 긴 문자열을 생성할 때 서식 문자열을 사용하면 더 읽기 쉽게 만들 수 있음
- 하나의 행에서 `+` 연산자를 서로 다른 시맨틱으로 사용하면 전체적으로 무엇을 하는 코드인지 알기 어려움

### 개선 후 코드

```java
class Mission {

    Logbook logbook;
    LocalDate start;

    void update(String author, String message) {
        final LocalDate today = LocalDate.now();
        String entry = String.format("%S: [%tm-%<te-%<tY](Day %d)> %s%n",
                author, today,
                ChronoUnit.DAYS.between(start, today) + 1, message);
        logbook.write(entry);
    }
}
```

- **서식 문자열**로 해결
- `String.format()`이나 `System.out.printf(`)와 같은 포맷 메서드는 위치 지정자 문자가 포함된 데이터를 String 뒤에 나열한 순서대로 받아들임
- 최종적으로 무엇을 출력할지는 알기 어렵지만 이 방식은 문서화가 잘 된 표준이자 어수선한 코드를 해결할 훌륭한 대안임

## 2.8 직접 만들지 말고 자바 API 사용하기

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

- API에 있는 기능을 다시 구현하지 말고 재사용해야 함

> `Objects.requireNonNull` 은 정말 자주 사용된다


```java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    int getQuantity(Supply supply) {
        Objects.requireNonNull(supply, "supply must not be null");

        return Collections.frequency(supplies, supply);
    }
}
```

- `Collections.frequency` : Collections는 컬렉션 내 객체 출현 횟수를 세는 frequency() 메서드를 제공함
- `Objects.requireNonNull` : Objects 유틸리티 클래스의 requireNonNull()은 객체가 널이면 메시지와 함께 NullPointerException을 던짐
- API를 알면 코드의 문제를 훨씬 더 간단하게 해결할 수 있음
- 직접 작성한 코드는 API보다 버그를 일으킬 가능성이 큼
