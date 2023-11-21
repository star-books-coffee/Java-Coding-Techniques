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
