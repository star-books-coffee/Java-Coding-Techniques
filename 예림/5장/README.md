# 5장. 문제 발생에 대비하기
## 5.1 빠른 실패

### 개선 전 코드

```java
class CruiseControl {
	static final double SPEED_OF_LIGHT_KMH = 1079252850;
	static final double SPEED_LIMIT = SPEED_OF_LIGHT_KMH;

	private double targetSpeedkmh;
	
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

- 메서드의 정상적인 제어 흐름이 눈에 들어오지 않음 (두 번째가 정상 제어 흐름인데 앞 뒤로 예외 처리 분기에 둘러싸여 있음)
- 조건 분기가 서로 연결되어 있다보니 모든 조건을 함께 이해해야 함

### 개선 후 코드

```java
class CruiseControl {
	static final double SPEED_OF_LIGHT_KMH = 1079252850;
	static final double SPEED_LIMIT = SPEED_OF_LIGHT_KMH;

	private double targetSpeedkmh;
	
	void setTargetSpeedKmh(double speedKmh) {
		if (speedKmh < 0 || speedKmh > SPEED_LIMIT) {
			throw new IllegalArgumentException();
		}
		targetSpeedKmh = speedKmh;
	}
}
```

- 매개변수 검증과 일반적인 경로를 분리하고 나머지 두 조건을 합쳐 메서드 상단에 둠
- 다시 말해 메서드가 빠르게 실패함

## 5.2 항상 가장 구체적인 예외 잡기

### 개선 전 코드

```java
class TransmissionParser {
	static Transmission parse(String rawMessage) {
		if (rawMessage != null) {
						&& rawMessage.length() != Transmission.MESSAGE_LENGTH) {
			throw new IllegalArgumentException("Bad message received!");
		}
	
		String rawId = rawMessage.substring(0, Transmission.ID_LENGTH);
		String rawContent = rawMessage.substring(Transmission.ID_LENGTH);
		try {
			int id = Integer.parseInt(rawId);
			String content = rawContent.trim();
			return new Transmission(id, content);
		} catch (Exception e) {
				throw new IllegalArgumentException("Bad message received");
		}
	}
}
```

- 자바의 예외를 잡으려면 항상 구체적인 예외 타입을 잡아야 함
- 예외를 잡아 버그를 숨겼다고 해서 버그를 고쳤다고 할 수는 없음 (때가 되면 더 불편한 시점에 프로그램이 실패함)

### 개선 후 코드

```java
class TransmissionParser {
	static Transmission parse(String rawMessage) {
		if (rawMessage != null) {
						&& rawMessage.length() != Transmission.MESSAGE_LENGTH) {
			throw new IllegalArgumentException("Bad message received!");
		}
	
		String rawId = rawMessage.substring(0, Transmission.ID_LENGTH);
		String rawContent = rawMessage.substring(Transmission.ID_LENGTH);
		try {
			int id = Integer.parseInt(rawId);
			String content = rawContent.trim();
			return new Transmission(id, content);
		} catch (NumberFormatException e) {
				throw new IllegalArgumentException("Bad message received");
		}
	}
}
```

- 이제 NullPointerException과 같은, 받아서는 안 될 오류를 더 이상 받지 않음
- 구체적인 예외를 잡기 위해 코드가 늘어나더라도 버그가 많은 코드보다 훨씬 나음
- 어떤 방법으로 캐치 블록을 조직하든 가장 구체적인 예외를 잡는 것이 중요함

## 5.3 메시지로 원인 설명

### 개선 전 코드

```java
class TransmissionParser {
	static Transmission parse(String rawMessage) {
		if (rawMessage != null) {
						&& rawMessage.length() != Transmission.MESSAGE_LENGTH) {
			**throw new IllegalArgumentException();**
		}
	
		String rawId = rawMessage.substring(0, Transmission.ID_LENGTH);
		String rawContent = rawMessage.substring(Transmission.ID_LENGTH);
		try {
			int id = Integer.parseInt(rawId);
			String content = rawContent.trim();
			return new Transmission(id, content);
		} catch (NumberFormatException e) {
				**throw new IllegalArgumentException("Bad message received");**
		}
	}
}
```

- 예외 처리는 예외를 잡는 것 뿐만 아니라 던지는 것까지 포함함. 예외를 던질 때는 타입 규칙을 지켜야 예외를 더 쉽게 처리할 수 있음
- 예외 자체로 자세한 맥락을 알 수 있다면 예외 스택 추적을 기반으로 근본 원인을 찾기 쉬움. 그러나 위 코드에서 던지는 예외는 맥락이 부족함
    - 첫 번째 IllegalArgumentException은 기본 생성자로 생성하고 아무 맥락도 제공하지 않음
    - 두 번째 IllegalArgumentException은 메시지를 입력으로 넣어도 도움이 안됨

### 개선 후 코드

```java
class TransmissionParser {
	static Transmission parse(String rawMessage) {
		if (rawMessage != null) {
						&& rawMessage.length() != Transmission.MESSAGE_LENGTH) {
			throw new IllegalArgumentException(
					String.format("Expected %d, but got %d characters in '%s'", 
							Transmission.MESSAGE_LENGTH, rawMessage.length(),
							rawMessage));
		}
	
		String rawId = rawMessage.substring(0, Transmission.ID_LENGTH);
		String rawContent = rawMessage.substring(Transmission.ID_LENGTH);
		try {
			int id = Integer.parseInt(rawId);
			String content = rawContent.trim();
			return new Transmission(id, content);
		} catch (NumberFormatException e) {
				throw new IllegalArgumentException(
						String.format("Expected number, but got '%s' in '%s'",
										 rawId, rawMessage));
		}
	}
}
```

- 바라는 것, 받은 것, 전체 맥락 세 가지를 골고루 제공 → 근본적인 원인 파악이 쉬움
- 예외를 일으킨 상황을 더 쉽게 재현 가능
- 세 가지 정보를 테스트 케이스로 재사용 가능 (JUnit 테스트로 변환해 버그 픽스를 만들고 이후 회귀 테스트로 쓰면 됨)
- 2.7절에서 사용했던 `[EXPECTED], but got [ACTUAL] in [CONTEXT]` 형태의 템플릿 사용

## 5.4 원인 사슬 깨지 않기

### 개선 전 코드

```java
class TransmissionParser {
	static Transmission parse(String rawMessage) {
		if (rawMessage != null) {
						&& rawMessage.length() != Transmission.MESSAGE_LENGTH) {
			throw new IllegalArgumentException(
					String.format("Expected %d, but got %d characters in '%s'", 
							Transmission.MESSAGE_LENGTH, rawMessage.length(),
							rawMessage));
		}

		String rawId = rawMessage.substring(0, Transmission.ID_LENGTH);
		String rawContent = rawMessage.substring(Transmission.ID_LENGTH);
		try {
			int id = Integer.parseInt(rawId);
			String content = rawContent.trim();
			return new Transmission(id, content);
		} catch (NumberFormatException e) {
				throw new IllegalArgumentException(
						String.format("Expected number, but got '%s' in '%s'",
										 rawId, rawMessage));
		}
	}
}
```

- 예외를 잡았지만 처리할 수 없다면 반드시 다시 던져야 함
    - 예외에 버그가 있을 경우 프로그램이 충돌할 때까지 전달될 수 있도록 하기 위해서
- 버그를 추적할 때 원인 사슬이 자세하면 엄청난 도움이 되므로 사슬이 끊기지 않도록 해야함
    - 원인 사슬 : 각 예외가 그 예외를 일으킨 예외와 연결된 리스트를 보여주는 스택 추적
- 위 코드는 NumberFormatException을 잡고 새 IllegalArgumentException을 던지므로 원인 사슬이 깨짐

### 개선 후 코드

```java
class TransmissionParser {
	static Transmission parse(String rawMessage) {

		... (생략) ...

		try {
			... (생략) ...
		} catch (NumberFormatException e) {
				**throw new IllegalArgumentException(
						String.format("Expected number, but got '%s' in '%s'",
										 rawId, rawMessage), e);**
		}
	}
}
```

- 예외에는 다양한 생성자가 있고 그 중에는 원인으로 Throwable을 전달하는 생성자도 있음. Throwable을 전달함으로써 예외와 원인을 연관시키고 원인 사슬이 만들어짐
- `Exception(String message, Throwable cause)` 생성자를 사용하고 message도 함께 제공하기
- **원인 사슬이 깨지는 최악의 예시**
    
    ```java
    } catch (NumberFormatException e) {
    	// 이런! 원인 사슬이 끊겼네요!
    	throw new IllegalArgumentException(e.getCause());
    }
    ```
    
    - NumberFormatException 이 throw에서 빠짐으로써 원인만 연결되고 예외 자체가 연결되지는 못함.
- **위 코드 개선하기**
    
    ```java
    throw new IllegalArgumentException("Message", e);
    ```
    
    - message와 잡았던 예외를 즉시 원인으로 전달하기

## 5.5 변수로 원인 노출

### 개선 전 코드

```java
class TransmissionParser {
	static Transmission parse(String rawMessage) {
		if (rawMessage != null) {
						&& rawMessage.length() != Transmission.MESSAGE_LENGTH) {
			throw new IllegalArgumentException(
					String.format("Expected %d, but got %d characters in '%s'", 
							Transmission.MESSAGE_LENGTH, rawMessage.length(),
							rawMessage));
		}

		String rawId = rawMessage.substring(0, Transmission.ID_LENGTH);
		String rawContent = rawMessage.substring(Transmission.ID_LENGTH);
		try {
			int id = Integer.parseInt(rawId);
			String content = rawContent.trim();
			return new Transmission(id, content);
		} catch (NumberFormatException e) {
				throw new IllegalArgumentException(
						String.format("Expected number, but got '%s' in '%s'",
										 rawId, rawMessage), e);
		}
	}
}
```

- 예외는 단순히 표준 클래스가 아니라 필드와 메서드, 생성자를 가질 수 있는 클래스이므로 이러한 구조를 이용해 기계가 읽을 수 있는 방식으로 예외의 원인을 나타낼 수 있음
- 위 코드에는 1) 중복 코드와 2) 감추어진 정보라는 문제가 있음
    - 1) IllegalArgumentException의 message에 두 번이나 “in %s”라는 방식으로 rawMessage를 넣고 있음
    - 2) rawMessage를 예외 message에 인코딩함 → 최종 사용자에게 어떤 종류의 메시지가 오류를 일으켰는지 알리고 싶을 때 추출하기 어려움

### 개선 후 코드

```java
class TransmissionParser {
	static Transmission parse(String rawMessage) {
		if (rawMessage != null) {
						&& rawMessage.length() != Transmission.MESSAGE_LENGTH) {
			**throw new MalformedMessageException(**
					String.format("Expected %d, but got %d characters in '%s'", 
							Transmission.MESSAGE_LENGTH, rawMessage.length(),
							rawMessage));
		}

		String rawId = rawMessage.substring(0, Transmission.ID_LENGTH);
		String rawContent = rawMessage.substring(Transmission.ID_LENGTH);
		try {
			int id = Integer.parseInt(rawId);
			String content = rawContent.trim();
			return new Transmission(id, content);
		} catch (NumberFormatException e) {
				throw new **MalformedMessageException**(
						String.format("Expected number, but got '%s'",
										 rawId, rawMessage), e);
		}
	}
}
final class MalformedMessageException extends IllegalArguementException {
	final String raw;
	
	MalformedMessageException(String message, String raw) {
		super(String.format("%s in '%s'", message, raw));
		this.raw = raw;
	}

	MalformedMessageException(String message, String raw, Throwable cause) {
		super(String.format("%s in '%s'", message, raw), cause);
		this.raw = raw;
	}
	
}
```

- raw 메시지 필드가 들어간 `MalformedMessageException`을 정의하고 사용하면 됨
- 향후 최종 사용자 정보를 자세히 알고 싶거나 예외를 더 철저히 처리하고 싶을 때 raw 필드 쉽게 추출 가능
- 클래스와 그 필드를 final로 선언해 맞춤형 예외를 불변으로 만들면 됨

## 5.6 타입 변환 전에 항상 타입 검증하기

### 개선 전 코드

```java
class Network {
	
	ObjectInputStream inputStream;
	Intercom interCom;
	
	void listen() throws IOException, ClassNotFoundException {
		while (true) {
			Object signal = inputStream.readObject();
			CrewMessage crewMessage = (CrewMessage) signal;
			interCom.broadcast(crewMessage);
		}
	}
}
```

- 프로그램에서 동적 객체를 사용하려면 명시적으로 어떤 타입이든 변환(casting)해야함
- 위 예제는 무사히 컴파일되고 스트림 내 객체 타입과 일치하면 순조롭게 진행된다
- 하지만 메서드는 스트림에 실제로 어떤 타입이 들어올지 제어할 수 없다 → CrewMessage가 아닌 객체가 들어오면 ClassCastException을 일으키며 충돌
- 예외를 잡을 수 있더라도 하면 안되는 이유는 전형적으로 수정 가능한 코드 내 버그를 알려주기 때문임
- 그러니 잡는 대신 코드를 고쳐야 함

### 개선 후 코드

```java
class Network {

    ObjectInputStream inputStream;
    InterCom interCom;

    void listen() throws IOException, ClassNotFoundException {
        while (true) {
            Object signal = inputStream.readObject();
            if (signal **instanceof** CrewMessage) {
                CrewMessage crewMessage = (CrewMessage) signal;
                interCom.broadcast(crewMessage);
            }
        }
    }
}
```

- instanceof 연산자로 타입을 검증하고 signal을 읽는다
- 코드 몇 줄이 늘어나도 자바에서 ClassCastException을 피할 방법은 없음
- 여러 타입으로 된 집합, 예제로 치자면 다양한 타입으로 된 메시지를 받고 싶다면 instanceof로 차례대로 검증해야 함
- 프로그램이 외부와 상호작용할 때는 항상 예상하지 못한 입력을 처리할 수 있도록 대비해야 함

## 5.7 항상 자원 닫기

### 개선 전 코드

```java
class Logbook {

    static final Path LOG_FOLDER = Paths.get("/var/log");
    static final String FILE_FILTER = "*.log";

    List<Path> getLogs() throws IOException {
        List<Path> result = new ArrayList<>();

        DirectoryStream<Path> directoryStream =
                Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER);
        for (Path logFile : directoryStream) {
            result.add(logFile);
        }
        directoryStream.close();

        return result;
    }
}
```

- 더 이상 자원이 필요없으면 바로 해제해야 한다
- 위 예제에서 close()로 자원을 해제하기 전에 자원을 사용하다가 예외가 발생하면 자원은 프로그램이 종료될 때까지 해제되지 못함 → **자원 누출**
- 나중에 프로그램이 같은 자원을 다시 요청하면 프로그램 자체에도 문제가 생김
- 어떻게 코드에서 사용했던 자원을 항상 닫을 수 있을까?

### 개선 후 코드

```java
class Logbook {

    static final Path LOG_FOLDER = Paths.get("/var/log");
    static final String FILE_FILTER = "*.log";

    List<Path> getLogs() throws IOException {
        List<Path> result = new ArrayList<>();

        try (DirectoryStream<Path> directoryStream =
                     Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER)) {
            for (Path logFile : directoryStream) {
                result.add(logFile);
            }
        }

        return result;
    }
}
```

- 자바 7부터는 try-with-resources 구문으로 자원을 안전하고 고상하게 닫을 수 있음
- AutoCloseable 인터페이스를 구현한 클래스여야 동작
- try 블록이 끝나면 무슨 일이 있어도 자바가 알아서 close() 호출을 처리함
- try-with-resources는 단지 문법적으로 쓰기 편한 표현일 뿐 컴파일러는 아래처럼 확장함
    
    ```java
    DirectoryStream<Path> resource =
            Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER);
    try {
        // usage of resource
    } finally {
        if (resource != null) {
            resource.close();
        }
    }
    ```
    
    - finally 블록에서 자원을 닫되 null이 아닐 때만 닫음으로써 NullPointerException도 피함

## 5.8 항상 다수 자원 닫기

### 개선 전 코드

```java
class Logbook {

    static final Path LOG_FOLDER = Paths.get("/var/log");
    static final Path STATISTICS_CSV = LOG_FOLDER.resolve("stats.csv");
    static final String FILE_FILTER = "*.log";

    void createStatistics() throws IOException {
        DirectoryStream<Path> directoryStream =
                Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER);
        BufferedWriter writer =
                Files.newBufferedWriter(STATISTICS_CSV);

        try {
            for (Path logFile : directoryStream) {
                final String csvLine = String.format("%s,%d,%s",
                        logFile,
                        Files.size(logFile),
                        Files.getLastModifiedTime(logFile));
                writer.write(csvLine);
                writer.newLine();
            }
        } finally {
            directoryStream.close();
            writer.close();
        }
    }
}
```

- try-with-resources 구문을 쓰지 않고 직접 자원을 닫기는 매우 어려움
- 위 예제는 5.7에서 보았던 직접 구현 버전과 거의 비슷하지만 예외 때문에 일이 어긋날 수 있음
- writer 생성에 실패하면 메서드가 예외와 함께 종료되어 directoryStream이 닫히지 않음
- directoryStream 닫기에서 예외가 발생하면 writer가 닫히지 않음

### 개선 후 코드

```java
class Logbook {

    static final Path LOG_FOLDER = Paths.get("/var/log");
    static final Path STATISTICS_CSV = LOG_FOLDER.resolve("stats.csv");
    static final String FILE_FILTER = "*.log";

    void createStatistics() throws IOException {
        **try (DirectoryStream<Path> directoryStream =**
                     Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER)**;**
             BufferedWriter writer =
                     Files.newBufferedWriter(STATISTICS_CSV)) {
            for (Path logFile : directoryStream) {
                String csvLine = String.format("%s,%d,%s",
                        logFile,
                        Files.size(logFile),
                        Files.getLastModifiedTime(logFile));
                writer.write(csvLine);
                writer.newLine();
            }
        }
    }
}
```

- try-with-resources 블록은 동시에 여러 자원을 처리할 수 있음
- 여러 자원을 사용하고 싶다면 try-with-resources 안에서 세미콜론으로 구분해 주기만 하면 됨

```java
try (open resource1; open resource2) {
	// 자원 사용
}
```

- 내부적으로 컴파일러는 try-with-resources 블록 내 각 자원을 확장해 여러 중첩 블록을 생성함
- 자원을 직접 관리하지 말고 try-with-resources 블록 안에서 자원을 열어라

## 5.9 빈 catch 블록 설명하기

### 개선 전 코드

```java
class Logbook {

    static final Path LOG_FOLDER = Paths.get("/var/log");
    static final String FILE_FILTER = "*.log";

    List<Path> getLogs() throws IOException {
        List<Path> result = new ArrayList<>();

        try (DirectoryStream<Path> directoryStream =
                     Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER)) {
            for (Path logFile : directoryStream) {
                result.add(logFile);
            }
        } catch (NotDirectoryException e) {

        }

        return result;
    }
}
```

- 예외는 예외를 의미 있게 처리할 수 있을 때만 잡아야 함
- catch 블록이 왜 비어 있는지 아무 힌트도 없으면 빈 catch 블록은 무조건 버그처럼 보임
- 그러면 의도를 어떻게 설명하면 좋을까?

### 개선 후 코드

```java
class Logbook {

    static final Path LOG_FOLDER = Paths.get("/var/log");
    static final String FILE_FILTER = "*.log";

    List<Path> getLogs() throws IOException {
        List<Path> result = new ArrayList<>();

        try (DirectoryStream<Path> directoryStream =
                     Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER)) {
            for (Path logFile : directoryStream) {
                result.add(logFile);
            }
        } catch (NotDirectoryException ignored) {
            // 디렉터리가 없으면 -> 로그도 없다!
        }

        return result;
    }
}
```

- 예외 변수명을 e에서 ignored로 바꿈 → 예외를 무시하겠다고 명시적으로 드러냄
- 예외를 왜 무시하는지 주석을 추가함
- CONDITION → EFFECT 템플릿
    - CONDITION : 왜 예외를 던졌는지
    - EFFECT : 왜 그냥 넘기고 무시할 수 있는지
- 예외 변수는 흔한 e보다 더 나은 이름을 짓는 게 좋음