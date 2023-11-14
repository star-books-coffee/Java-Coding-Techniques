## 1장. 우선 정리부터

### 1.1. 쓸모없는 비교 피하기

- boolean 반환값을 명시적으로 비교하는 코드는 안티 패턴이므로 제거하라

```java
if(eunju.isHuman() == true)
```

- 메서드 내의 단일 반환문 vs. 다중 반환문
    - 메서드를 일찍 종료하고 싶은 경우, 코드가 더 적게 드는 다중 반환문 사용

### 1.2. 부정 피하기

- 코드에서는 긍정 표현식이 부정 표현식보다 낫다

### 1.3. 불 표현식을 직접 반환

```java
boolean isValid() {
	if(missions < 0 || name == null || name.trim().isEmpty()) {
		return false;
	}
	else {
		return true;
	}
}
```

```java
boolean isValid() {
	return missions >=0 && name != null && !name.trim().isEmpty();
}
```

- 드모르간의 법칙을 활용하라
    - !A && !B = ! (A || B)
    - !A || !B == ! (A && B)

### 1.4. 불 표현식 간소화

- 여러 조건문이 합쳐진 불 표현식은 조건의 의미에 따라 그루핑하여 나눠라
    - 변수명, 메서드명으로 더 이해하기 쉽게 표현해라

### 1.5. 조건문에서 NullPointerException 피하기

- 반드시 null 을 먼저 확인한 후, **`도메인에 따라`** “유효하지 않은” 값을 검사 해야 한다
    - 빈 문자열, 빈 리스트 검사 후 특정 값 확인

```java
void writeMessage(String message, Path location) {
	if (Files.isDirectory(location)) {...}
	if (message.trim().equals("") || message == null) {...}
}
```

```java
void writeMessage(String message, Path location) {
	if (message == null || message.trim().equals("")) {...}
	// message 값이 없으면 Files 에 write 할 작업을 할 수 없으므로 isDirectory 도 수행할 필요가 없음
	if (location == null || Files.isDirectory(location)) {...}
}
```

- 매개변수 검사는 public, protected, default 메서드에서만 하면 됨

### 1.6. 스위치 실패 피하기

```java
void authorize(User user){
	switch (user.getRank()) {
		case UNKNOWN: 
			cruiseControl.logUnthorizedAccessAttempt(); // break 문이 없는 경우
		case ASTRONAUT:
			cruiseControl.grantAccess(user);
			break;
	}
}
```

- break 문을 빼먹는 경우, 의도치 못한 결과가 나올 수 있음 ( break 문 또는 블록 끝에 다다라야 실행이 멈추기 때문 )
- **의도적으로 break 누락했다면 반드시 주석을 남겨라**
- 코딩하지 않은 값을 명시적으로 처리하는 예비 분기문이 항상 있어야 한다 ( switch 문은 default 케이스로 이러한 기능을 제공 )

### 1.7. 항상 괄호 사용하기

- if 문 조건이 한 줄인 경우, 중괄호를 안쓰는 경우가 많은데 항상 습관처럼 사용하는 게 버그를 예방하는 훌륭한 대비책이다
- 새로운 코드도 안전하게 추가할 수 있고, 코드를 더 읽기 쉽게 만들어준다

### 1.8. 코드 대칭 이루기

- 조건과 명령문이 여러 개면 한 번에 읽고 이해하기가 어려운데, **`조건 분기를 대칭적 방법으로 구조화`** 하면 코드를 쉽게 이해할 수 있다.
- 아래 코드는 “코드 대칭성의 부재” 이라는 문제를 가진다 (if-else 문으로 연결되어 있어 접근 거절/부여 코드가 합쳐져 있음)

```java
class BoardComputer {
    void authorize(User user) {
        Objects.requireNonNull(user);

        if (user.isUnknown()) {
            cruiseControl.logUnauthorizedAccessAttempt();
        } else if (user.isAstronaut()) {
            cruiseControl.grantAccess(user);
        } else if (user.isCommander()) {
            cruiseControl.grantAccess(user);
            cruiseControl.grantAdminAccess(user);
        }
    }
}
```

- 두 코드를 서로 다른 코드 블록으로 분리하면, 코드 대칭성을 향상시킬 수 있음
    - 서로 다른 접근 유형을 별개의 if 문으로 묶음 : 접근 거절 if 문 / 접근 부여 if-else 문 코드 분리되어 있음

```java
class BoardComputer {
    CruiseControl cruiseControl;
    void authorize(User user) {
        Objects.requireNonNull(user);
			
	// 접근 거절
        if (user.isUnknown()) { 
            cruiseControl.logUnauthorizedAccessAttempt();
            return;
        }

	// 접근 부여
        if (user.isAstronaut()) {
            cruiseControl.grantAccess(user);
        } else if (user.isCommander()) {
            cruiseControl.grantAccess(user);
            cruiseControl.grantAdminAccess(user);
        }
    }
}
```

### 1.9. 1장에서 배운 내용

- 코드 가독성을 높이기 위해 노력하라
