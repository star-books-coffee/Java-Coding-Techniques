# 1장. 우선 정리부터
## 1.1 쓸모 없는 비교 피하기

> 유지보수가 쉬운 코드 개발은 단순한 선행 그 이상입니다. 바로 전문가가 되는 과정이자 기술을 단련하는 과정입니다.
> 

### 개선 전 코드

```java
class Laboratory {

	Microscope microscope;

	Result analyze(Sample sample) {
		if (microscope.isInorganic(sample) == true) {
			return Result.INORGANIC;
		} else {
			return analyzeOrganic(sample);
		}
	}

	private Result analyzeOrganic(Sample sample) {
		if (microscope.isHumanoid(sample) == false) {
			return Result.ALIEN;
		} else {
			return Result.HUMANOID;
		}
	}
}
```

- 불 반환값과 불 원시 타입(true와 false)을 명시적으로 비교하는 안티 패턴임
- **불 표현식은 불 원시값과 비교하지 않아도 됨** (코드를 어수선하고 읽기 어렵게 만들기 때문)

### 개선 후 코드

```java
class Laboratory {

	Microscope microscope;	

	Result analyze(Sample sample) {
		if (microscope.isInorganic(sample)) {
			return Result.INORGANIC;
		} else {
			return analyzeOrganic(sample);
		}
	}

	private Result analyzeOrganic(Sample sample) {
		if (!microscope.isHumanoid(sample)) {
			return Result.ALIEN;
		} else {
			return Result.HUMANOID;
		}
	}
}
```

## 1.2 부정 피하기

- **코드에서는 긍정 표현식이 부정 표현식보다 더 낫다** (이해하기 더 쉽고 공간도 덜 차지하기 때문 / 부정 표현은 추가적인 사고과정이 더 필요함)

### 개선 후 코드

```java
if (microscope.isOrganic(sample)) {
	return analyzeOrganic(sample);
} else {
	return Result.INORGANIC;
}
```

```java
if (microscope.isHumanoid(sample)) {
	return Result.HUMANOID;
} else {
	return Result.ALIEN;
}
```

- if와 else 블록 본문을 서로 바꿈
- 호출하려는 코드가 없더라도 제어할 수만 있다면 적절한 클래스에 메서드를 추가하는 것을 망설이지 마라
- 또, 부정적 메서드는 모두 제거하는 것이 가장 좋음. 비슷한 메서드 두개를 굳이 유지하지 마라
- 메서드를 추가하면 코드 중복을 줄이고 프로그램의 다른 부분에서도 재사용할 수 있으니 결국 코드가 줄어들게 됨

## 1.3 불 표현식을 직접 반환

### 개선 전 코드

```java
class Astronaut {
	
	String name;
	int missions;

	boolean isValid() {
		if (missions < 0 || name == null || name.trim().isEmpty()) {
			return false;
		} else {
			return true;
		}
	}
}
```

- if문이 실제 의미를 흐리기만 하는 지저분한 코드
- 메서드 반환 타입을 불 형태로 하면 됨

### 개선 후 코드 (1)

```java
class Astronaut {
	String name;
	int missions;

	boolean isValid() {
		return missions < 0 || name == null || name.trim().isEmpty();
	}
}
```

- 드모르간의 법칙을 적용해 조건문을 부정함
- 조건문이 이보다 더 복잡할 수 있으므로 조건문을 **더 작은 덩어리로 분할**하는 방향을 고려해야 함

### 개선 후 코드 (2)

```java
boolean isValid() {
	boolean isValidMissions = missions >= 0;
	boolean isValidName = name != null && !name.trim().isEmpty();
	return isValidMissions && isValidName;
}
```

- 조건문을 세 개 이상 합칠 때는 위와 같은 간소화를 고려 해야 함
- 반환 타입이 불일 때만 동작하는 해법임
- 조건문 덩어리를 다른 곳에서도 써야 한다면 1.4절을 참조

## 1.4 불 표현식 간소화

### 개선 전 코드

```java
class SpaceShip {

	Crew crew;
	FuelTank fuelTank;
	Hull hull;
	Navigator navigator;
	OxygenTank oxygenTank;

	boolean willCrewSurvive() {
		return hull.holes == 0 &&
			fuelTank.fuel >= navigator.requiredFuelToEarth() &&
			oxygenTank.lastsFor(crew.size) > navigator.timeToEarth();
	}
}
```

- 1.3절에서 배운 방법대로 적용해 반환문 하나로 압축한 상태
- 여러 조건문을 하나로 합쳐 확인해야 한다면 다른 식으로 묶는 것이 더 나음
- 한 메서드 안에서는 추상화 수준이 비슷하도록 명령문을 합쳐야 함

> **유용한 괄호**
> 
> 
> 불 조건은 특히 괄호가 없으면 다루기 까다로움. 많은 개발자가 불 연산자 우선순위를 따로 기억하지 못함
> 

### 개선 후 코드

```java
class SpaceShip {
	
	...(생략)....

	boolean willCrewSurvive() {
		boolean hasEnoughResources = hasEnoughFuel() && hasEnoughOxygen();
		return hull.isIntact() && hasEnoughResources;
	}

	private boolean hasEnoughOxygen() {
		return oxygenTank.lastsFor(crew.size) > navigator.timeToEarth();
	}
	
	private boolean hasEnoughFuel() {
		return fuelTank.fuel >= navigator.requiredFuelToEarth();
	}
}
```

- 이제 다른 메서드를 호출해 반환값을 집계함

## 1.5 조건문에서 NullPointerException 피하기

### 개선 전 코드

```java
class Logbook {
	
	void writeMessage(String message, Path location) throws IOException {
		if (Files.isDirectory(location)) {
			throw new IllegalArgumentException("The path is invalid!");
		}
		if (message.trim().equals("") || message == null) {
			throw new IllegalArgumentException("The message is invalid!");
		}
		String entry = LocalDate.now() + ": " + message;
		Files.write(location, Collections.singletonList(entry),
				StandardCharsets.UTF_8, StandardOpenOption.CREATE,
				StandardOpenOption.APPEND);
	}
}
```

- `NullPointerException` : null을 참조하는 메서드를 호출하거나 속성에 접근할 때 발생 ⇒ 막기 위해선 유효성 검사 필요
- 위 코드의 문제점
    - locationdl null이면 Files.isDirectory()는 별다른 설명 없이 NullPointException과 함께 실패
    - message가 null이면 message.equals(””)를 먼저 확인하게 되므로 마찬가지로 NullPointException과 함께 실패
- 인수를 검증할 때는 반드시 null을 먼저 확인한 후 도메인에 따라 “유효하지 않은” 값을 검사해야 함
- 메서드 인수로 null을 전달하는 방식은 메서드가 매개변수 없이도 올바르게 기능한다는 뜻이므로 피하는 것이 좋음
- 꼭 해야한다면 매개변수가 있는 메서드와 없는 메서드 두 개로 리팩터링하는 게 좋음

### 개선 후 코드

```java
class Logbook {
	
	void writeMessage(String message, Path location) throws IOException {
		if (message == null || message.trim().isEmpty()) {
			throw new IllegalArgumentException("The path is invalid!");
		}
		if (location == null || Files.isDirectory(location)) {
			throw new IllegalArgumentException("The message is invalid!");
		}
		String entry = LocalDate.now() + ": " + message;
		Files.write(location, Collections.singletonList(entry),
				StandardCharsets.UTF_8, StandardOpenOption.CREATE,
				StandardOpenOption.APPEND);
	}
}
```

- 모든 인수에 대해 null 값 여부 확인
- 메서드 서명 내 인수 순서에 따라 확인
- 내장 메서드를 사용해 빈 문자열인지 확인
- 매개변수 검사는 public과 protected, default 메서드에서만 하면 됨 (이러한 메서드는 코드 어디서든 접근할 수 있고 접근이 어떻게 일어나는지 제어가 어렵기 때문)

## 1.6 스위치 실패 피하기

- break문을 빠뜨려 야기된 버그가 많음
- 간혹 의도적으로 break를 누락했다면 반드시 주석을 남기는 것이 좋음
- 서로 다른 관심사는 서로 다른 코드 블록에 넣어야 하지만, switch 문은 관심사를 분리하기 어려움
- if문 사용을 선호

## 1.7 항상 괄호 사용하기

- 범위를 정의하는 중괄호를 빼먹으면 아무도 버그를 못 알아챌 수 있으므로 항상 괄호를 사용하는 것이 좋음

## 1.8 코드 대칭 이루기

- 조건 분기를 대칭적 방법으로 구조화하면 코드를 쉽게 이해하고 파악할 수 있음

### 개선 전 코드

```java
class BoardComputer {
	
	CruiseControl cruiseControl;

	void authorize(User user) {
		Objects.requiredNonNull(user);
		if (user.isUnknown()) {
			cruiseControl.logUnauthroizedAccessAttempt();
		} else if (user.isAstronaut()) {
			cruiseControl.grandAccess(user);
		} else if (user.isCommandar()) {
			cruiseControl.grantAccess(user);
			cruiseControl.grantAdminAccess(user);
		}
	}
}
```

- 위 예시는 조건과 명령문이 계속 연이어 나옴
- 모든 분기가 비슷한 관심사를 표현하지도 않고, 병렬구조를 띠지도 않으며, 세 가지 분기 모두 대칭도 아님
    - 첫 번째 분기는 접근을 거절, 두 번째, 세 번째 분기는 접근을 부여

```java
class BoardComputer {
	
	CruiseControl cruiseControl;

	void authorize(User user) {
		Objects.requiredNonNull(user);
		if (user.isUnknown()) {
			cruiseControl.logUnauthroizedAccessAttempt();
			return;
		}

		if (user.isAstronaut()) {
			cruiseControl.grandAccess(user);
		} else if (user.isCommandar()) {
			cruiseControl.grantAccess(user);
			cruiseControl.grantAdminAccess(user);
		}
	}
}
```

- 권한을 부여하는 코드와 권한을 부여하지 않는 코드를 분리해 코드 대칭성 향상
- 먼저 승인되지 않은 접근 처리 후 메서드 종료
- 최적화할 여지가 아직 남음. grantAccess를 동일한 인수로 호출하는 두 조건을 별개의 비공개 메서드로 추출 가능
