## 5장. 문제 발생에 대비하기
### 5.1.  빠른실패

```java
class CruiseControl {
    void setTargetSpeedKmh(double speedKmh) {
        if (speedKmh < 0) {
            throw new IllegalArgumentException();
        } else if (speedKmh <= SPEED_LIMIT) {
            targetSpeedKmh = speedKmh;
        } else {
            throw new IllegalArgumentException();
        }
    }
}
```

```java
class CruiseControl {
    void setTargetSpeedKmh(double speedKmh) {
        if (speedKmh < 0 || speedKmh > SPEED_LIMIT) {
            throw new IllegalArgumentException();
        }
        targetSpeedKmh = speedKmh;
    }
}
```

- **메서드를 빠르게 실패하도록 하라 (예외처리 로직을 먼저)**
- 매개변수 검증을 통한 예외처리를 우선적으로 하고, 일반적인 메서드 경로로 넘어갈 수 있게 구현하라

### 5.2. 항상 가장 구체적인 예외 잡기

- **예외를 잡으려면 가장 구체적인 예외 타입만 잡아라, 그렇지 않고 일반적인 타입을 잡으면 잡아선 안될 오류까지 잡힐 위험이 존재한다**
    - throwable 을 잡으면 OutOfMemoryError 와 같은 가상 머신 내 오류까지 잡힐 수 있음
    - Exception 대신 try 내 코드에서 던질 만한 가장 구체적 예외를 찾아라
- 가장 구체적인 예외를 잡기 위해 여러 예외를 잡아야 할 수도 있지만, 일반적 예외유형 잡는거보단 낫다

### 5.3. 메시지로 원인 설명

- 그냥 IllegalArgumentException() 이렇게 던지지 말고, **Exception 내에 메시지를 표시하라 (바라는 것, 받은 것, 전체 맥락)**
    - 세가지 정보는 나중에 `테스트 케이스` 로 재사용도 가능
 
### 5.7. 항상 자원닫기

```java
DirectoryStream<Path> directoryStream = Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER);

for (Path logFile : directoryStream) {
  result.add(logFile);
}

directoryStream.close();
```

- 더이상 자원이 필요없으면 바로 해제하면 된다
- 프로그램이 자원 연 후 close() 로 자원을 해제하기 전에, 예외가 발생하면 close() 가 실행되지 않아 프로그램 종료될 때까지 해제 X → `자원 누출`
    - 이후에 프로그램이 같은 자원 다시 요청시 프로그램 자체에도 문제 발생

```java
try (DirectoryStream<Path> directoryStream = Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER)) {
	for (Path logFile : directoryStream) {
	  result.add(logFile);
	}
}
```

- **try-with-resources 구문 사용하라**, AutoCloseable 인터페이스를 구현한 클래스여야 동작함
- try 블록 끝나면 무슨 일이 있어도 자바가 알아서 close() 호출 처리
- 컴파일러가 아래처럼 확장하여 동작
    - finally 블록에서 자원을 닫되, null 이 아닐 때만 닫음으로써 NPE 피함

```java
DirectoryStream<Path> directoryStream = Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER);
try {
	// 자원사용
} finally {
	if(resource != null) {
		resource.close();
	}
}
```

### 5.8. 항상 다수 자원닫기

```java
try (DirectoryStream<Path> directoryStream = Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER); BufferedWriter writer = Files.newBufferedWriter(STATISTICS_CSV)) { ... }
```

- **여러 자원을 사용하고 싶다면, try-with-resources 안에서 세미콜론으로 구분해줘라**
- 내부적으로 컴파일러가 아래처럼 확장하여 동작
    - try-with-resources 블록 내 각 자원을 확장해 여러 중첩 블록 생성

```java
// resource1 열기
try {
	// resource2 열기 
	try { 
		// resource1, resource2 사용
	} finally {
		resource2.close();
} finally {
	resource1.close();
}
```

### 5.9. 빈 catch 블록 설명하기

```java
try (DirectoryStream<Path> directoryStream = Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER)) {
		for (Path logFile : directoryStream) {
	    result.add(logFile);
    }
} catch (NotDirectoryException e) {

}
```

- **예외는 의미있게 처리할 수 있을 때만 잡아야 한다.**
- 빈 catch 블록은 무조건 버그처럼 보인다

```java
try (DirectoryStream<Path> directoryStream = Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER)) {
		for (Path logFile : directoryStream) {
	    result.add(logFile);
    }
} catch (NotDirectoryException ignored) {
// No directory -> no logs!
}
```

- **예외 변수명을 e → ignored 로 바꾸어, 예외를 무시하겠다고 명시적으로 드러내라**
- **예외를 왜 무시하는지 주석으로 추가하라**
