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
