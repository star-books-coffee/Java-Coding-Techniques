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
