## 4장. 올바르게 명명하기
### 4.1. 자바 명명 규칙 사용하기

- 자바 코드 규칙 : 자바 코드를 서식화하는 실질적 표준
- `class` : 대문자로 시작하는 CamelCase
- `상수` (final, static 인 변수) : 두드러지게 표현하기 위해 모든 철자가 대문자이고 용어를 밑줄로 구분
- 메서드, 필드 매개변수, 변수 : 첫 글자가 소문자로 시작하는 camelCase 의 변형
    - `메서드` : 동사로 명명
    - `변수` : 명사로 명명

### 4.2. 프레임워크에는 Getter/Setter 규칙 적용

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

- getter, setter 의 구조와 명명은 표준화가 잘되어 있어 여러 프레임워크에서 대부분 따름
    - hibernate : getter, setter 로 자바 인스턴스와 SQL 데이터베이스 내 행 변환
    - jackson : JSON 메시지에 사용
- getter, setter 만의 명세인 자바 빈(Java Bean) 명세도 따로 있음

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

- **필드 한정자를 private, getter/setter 를 public 로 변환하라**
- 반드시 자바 빈을 사용해야 하는 자바 프레임워크도 존재함

### 4.3. 한 글자로 명명하지 않기

- **변수를 한글자로 짓지 마라**
- 컴파일러는 컴파일타임에 이름을 대체하는데 바이트코드로는 전체이름과 한 글자 이름에 차이가 없으므로, 변수 이름이 길면 비효율적일까봐 걱정하지마라
