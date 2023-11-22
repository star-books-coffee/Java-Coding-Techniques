## 4.1 자바 명명 규칙 사용하기

- Class 명, 인터페이스명, enum 명은 대문자로 시작하는 CamelCase
- 상수는 모든 철자를 대문자, 용어를 밑줄(_)로 구분
- 메서드와 필드, 매개변수, 변수는 첫 글자가 소문자로 시작하는 camelCase의 변형

## 4.2 프레임워크에는 Getter/Setter 규칙 적용

### 개선 전 코드

```java
Astronaut {
	
	String name;
	boolean retired;

	Astronaut(String name) {
		this.name = name;
	}

	String getFullName() {
		return name;
	}

	void setFullName(String name) {
		this.name = name;
	}

	boolean getRetired(() {
		return retired;
	}
	
	void setRetiredState(boolean retired) {
		this.retired = retired;
	}
}
```

- 게터와 세터만의 명세인 자바 빈(Java Bean) 명세도 따로 있음
- 위 코드는 원치 않는 방식으로 동작할 수 있다

### 개선 후 코드

```java
class Astonaut {
	private String name;
	private boolean retired;

	public Astronaut() {
	}

	public Astronaut(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public boolean isRetired() {
		return retired;
	}

	public void setRetired(boolean retired) {
		this.retired = retired;
	}
}
```

- 위 코드가 유효한 자바 빈 클래스임
- 필드의 한정자를 private로, 게터와 세터는 public으로 바꿈
- 기본 생성자를 추가하고, 기본 생성자로 클래스의  빈 인스턴스를 만든 후 설정할 때는 세터를 호출해 필드에 값 할당
- 필드명을 넣어 게터와 세터를 다시 명명
    - boolean 필드는 세터명은 그대로지만 게터는 질문하듯이 isFoo()라고 명명
- 코드를 반드시 자바 빈으로 작성해야하는 것은 아니지만 반드시 자바 빈을 사용해야 하는 자바 프레임워크도 있음

## 4.3 한 글자로 명명하지 않기

- 단지 글자 하나로 의미를 전달하기는 힘듦
- `l`과 `1`, `O`와 `0`을 동시에 사용하지 말기

## 4.4 축약 쓰지 않기

- 코드를 더 빨리 이해하고 짜증도 덜 나려면 축약은 절대로 쓰지 않는 것이 나음 (ex. DIR, GLOB, bufW, lFile, csvLn 등)

### 개선 전 코드

```java
class Logbook {
  static final Path DIR = Paths.get("/var/log");
  static final Path CSV = DIR.resolve("stats.csv");
  static final String GLOB = "*.log";

  void createStats() throws IOException {
    try (DirectoryStream<Path> dirStr = Files.newDirectoryStream(DIR, GLOB);
       BufferedWriter bufW = Files.newBufferedWriter(CSV)) {
      for (Path lFile : dirStr) {
          String csvLn = String.format("%s,%d,%s",
                  lFile,
                  Files.size(lFile),
                  Files.getLastModifiedTime(lFile));
          bufW.write(csvLn);
          bufW.newLine();
      }
    }
  }
}
```

### 개선 후 코드

```java
class Logbook {
  static final Path LOG_FOLDER = Paths.get("/var/log");
  static final Path STATISTICS_CSV = DIR.resolve("stats.csv");
  static final String FILE_FILTER = "*.log";

  void createStats() throws IOException {
    try (DirectoryStream<Path> logs = Files.newDirectoryStream(LOG_FOLDER, FILE_FILTER);
       BufferedWriter writer = Files.newBufferedWriter(STATISTICS_CSV)) {
      for (Path log : logs) {
          String csvLn = String.format("%s,%d,%s",
                  log,
                  Files.size(log),
                  Files.getLastModifiedTime(log));
          bufW.write(csvLine);
          bufW.newLine();
      }
    }
  }
}
```

- bufW → writer (버퍼를 사용한다는 건 사실 별로 중요하지 않음)
- 가능하면 축약은 피하고 **매우** 일반적인 경우에만 사용하라
- 확신이 없으면 **풀어써라!**

## 4.5 무의미한 용어 쓰지 않기

### 개선 전 코드

```java
class MainSpaceShipManager {
	AbstractRocketPropulsionEngine abstractRocketPropulsionEngine;
	INavigationController navigationController;
	boolean turboEnabledFlag;
	
	void navigateSpaceShipTo(PlanetInfo planetInfo) {
		RouteData data = navigatinoController.calculateRouteData(planetInfo);
		LogHelper.logRouteData(data);
		abstractRocketPropulsionEngine.invokeTask(data, turboEnabledFlag);
	}
}
```

- 의미는 전혀 전달하지 않으면서 글자 수만 늘리는 단어는 제거해야 함
- `main`, `manager`, `data`, `info`, `flag`처럼 자주 쓰이는 무의미한 용어를 low-hanging fruits(가장 쉽게 해결할 수 있는 방법)이라고 부름
- `abstract`라는 용어나, `invoke`라는 메서드, 메서드명에 들어있는 매개변수 타입도 의미가 없음

```java
class spaceShip {
	Engine engine; 
	Navigator navigator;
	boolean turboEnabled;

	void navegateTo(Planet destination) {
		Route route = navigatinoController.calculateRouteTo(destination);
		Logger.log(route);
		engine.follow(route, turboEnabled);
```

- data, info, flag 용어 제거
- `abstract`나 `impl`, 인터페이스 명에 붙은 접두사 `I`도 불필요
- 클래스명이 SpaceShip이니 rocket과 같은 도메인 지정자도 없어도 됨
- logRouteDate()처럼 매개변수 종류를 메서드명에서 반복하는 경우도 제거 (매개변수 route가 어떤 데이터를 로깅하는지 알려줌)
- invoke, call, do 같은 동사, 패키지명에 종종 등장하는 misc, other, util과 같은 단어는 불필요함

## 4.6 도메인 용어 사용하기

- 개발 중인 대부분의 코드는 특정 도메인에 속하고 도메인마다 각기 어휘가 있음
- 프로그램에 해당하는 도메인 용어를 많이 넣을수록 코드는 점점 나아짐

> 이름은 모두 똑같이 중요할까? 아니다. 범위가 넓은 이름이 훨씬 더 많이 읽히므로 이름의 범위가 클수록 중요가 커짐
> 

### 개선 전 코드

```java
class Person {
	String lastName;
	String role;
	int travles;
	LocalDate employedSince;

	String serializeAsLine() {
		return String.join(",", 
							Arrays.asList(lastName,
											role,
											String.valueOf(travels),
											String.valueOf(employedSince))
		);
	}
}
```

- 클래스와 메서드가 매우 포괄적이고 왜 저 속성만 사용하는 건지, 출장횟수는 왜 기록하는 것인지 의도를 알 수 없음

### 개선 후 코드

```java
class Astronaut {
	String tagName;
	String rank;
	int missions;
	LocalDate activeDutySince;
	
	String toCSV() {
		return String.join(",", 
							Arrays.asList(tagName,
											rank,
											String.valueOf(missions),
											String.valueOf(activeDutySince))
		);
```

- 화성 탐사 작전을 클래스의 도메인으로 가정하니 다양한 속성의 의미를 이해하기가 쉬움
    - lastName → tagName : 우주비행사가 착용할 이름표에 넣을 이름
    - role → rank : 실제로 그 사람이 속하는 지위
    - travels →  missions : 우주에서 수행한 missions 횟수를 뜻한다는 게 명확해짐
- 화성 탐사 작전만 유일한 도메인인 것은 아니고, 코드는 기술적 개념도 다루므로 항상 암묵적으로 서로 다른 용어 집합을 갖는 기술적 도메인도 속함
    - serializeAsLine() → toCSV()
