# 6장. 올바르게 드러내기
## 6.1 Given-When-Then으로 테스트 구조화

### 개선 전 코드

```java
class CruiseControlTest {

    @Test
    void setPlanetarySpeedIs7667() {
        CruiseControl cruiseControl = new CruiseControl();
        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);
        Assertions.assertTrue(7667 == cruiseControl.getTargetSpeedKmh());
    }
}
```

- 메서드에 @Test 표기만 하면 JUnit이 테스트로 실행함
- 일반적으로 테스트는 given, when, then이라는 세 개의 핵심 부분으로 구성됨
- 모범 테스트 명세 예시
    - 2가 찍혀있는 계산기가 **주어졌을 때**(given)
    - 숫자 3을 더한 **경우(when)**
    - **그러면(then)** 숫자 5가 나타나야 한다
- given : 실제 테스트를 준비하는 단계이자 테스트하려는 기능을 실행하기 위한 전제 조건을 모두 포함함
- when : 실제로 테스트하려는 연산을 수행함
- then : when에서 수행한 결과가 실제로 기대했던 결과인지 명확히 드러냄(assertion)

### 개선 후 코드

```java
class CruiseControlTest {

    @Test
    void setPlanetarySpeedIs7667() {
        CruiseControl cruiseControl = new CruiseControl();

        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);

        Assertions.assertTrue(7667 == cruiseControl.getTargetSpeedKmh());
    }
}
```

- 주석까지 넣으면 구조가 확연히 드러남

```java
class Other {

    void otherTest() {
        // given
        CruiseControl cruiseControl = new CruiseControl();

        // when
        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);

        // then
        Assertions.assertTrue(7667 == cruiseControl.getTargetSpeedKmh());
    }
}
```

- 올바르게 구조화하는 데 능숙해지는 방법으로 주석 추가를 권하기도 함
- 하지만 일단 구조가 눈에 익으면 수직 빈 줄로도 충분함

## 6.2 의미 있는 어서션 사용하기

- 보통은 assertTrue()로 어서션을 작성하지 않음
- assertTrue()로 어서션을 작성하면 테스트가 실패할 경우 어떤 어서션이 실패했는지만 알 뿐 왜 실패했는지는 모름
    - java.lang.AssertionError라는 스택 추적을 받는데, 실패한 클래스, 실패한 어셔션의 줄 번호만 있고 아무 메시지도 없음
- 어서션은 두 값의 비교가 아니라 단지 불 값이 true여야 하는데 false인 것에만 신경 씀

### 개선 후 코드

```java
class CruiseControlTest {

    @Test
    void setPlanetarySpeedIs7667() {
        CruiseControl cruiseControl = new CruiseControl();

        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);

        Assertions.assertEquals(7667, cruiseControl.getTargetSpeedKmh());
    }
}
```

- 두 값이 같은지 확인하는 assertEquals()라는 다른 어서션을 사용
- 이 어서션을 사용하면 JUnit은 훨씬 더 나은 오류 메시지를 제공함
    
    ```java
    expected: <7667> but was <1337>
    ```
    
- assertTrue()로도 다양한 메시지를 제공할 수 있지만 다만 assertEquals()와 달리 테스트 실패 시 보여줄 메시지를 직접 만들어 assertTrue()에 넣어야 함
- 중요한 것은 더 나은 메시지를 얻으려면 검증하려는 테스트에 가장 적합한 어서션을 선택해야 한다는 점

## 6.3 실제 값보다 기대 값을 먼저 보이기

### 개선 전 코드

```java
class CruiseControlTest {

    @Test
    void setPlanetarySpeedIs7667() {
        CruiseControl cruiseControl = new CruiseControl();

        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);

        Assertions.assertEquals(cruiseControl.getTargetSpeedKmh(), 7667);
    }
}
```

- 테스트를 실행해 어서션에 실패하면 아래와 같은 메시지를 얻음
    
    ```java
    expected: <1337> but was <7667>
    ```
    
    - 메시지의 의미가 틀림. assertionEquals()의 인수가 잘못된 순서로 뒤바뀌었기 때문
- 실패한 테스트의 메시지를 읽을 때 최소한 그 메시지 자체는 무조건 옳다고 가정하므로 가정이 틀리면 안됨
- assertionEquals 순서에 주의를 기울여야 함

### 개선 후 코드

```java
class CruiseControlTest {

    @Test
    void setPlanetarySpeedIs7667() {
        CruiseControl cruiseControl = new CruiseControl();

        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);

        Assertions.assertEquals(7667, cruiseControl.getTargetSpeedKmh());
    }
}
```

- 실수를 하지 않으려면 **먼저** 무엇을 원하는지 생각하자
- 명확하고 설명적인 어서션 메시지는 필요할 때(테스트에 실패했을 때) 엄청난 도움이 됨
## 6.4 합당한 허용값 사용하기

- 부동소수점 연산을 테스트할 때는 소수점 자릿수를 명시해야 함
- 부동소수점 연산에 완전 일치는 없으므로 기대값과 실제값 사이의 약간의 차이를 허용해야 함
- JUnit에서 `assertEquals(double expected, double actual, double delta)` 어서션은 delta라는 허용값을 지원함

> 돈에는 부동소수점 연산을 하지 말고 대신에 long 변수에 저장하거나 BigDecimal을 이용해야 함
> 

## 6.5 예외 처리는 JUnit에 맡기기

- 테스트는 아무 예외도 던지지 않거나 특정 예외를 반드시 던지게 함
- JUnit에는 예외를 처리하는 내장 메커니즘이 있으므로 명시적으로 추가하지 말고 그냥 JUnit이 알아서 하게 놔두면 됨
    - try - catch 블록과 fail 호출 삭제
    - assertThrows 어셔션에 예외가 생기길 바라는 메서드만 전달하면 됨
        
        ```java
        Assertions.assertThrows(IOExcepetion.class, when);
        ```
        

## 6.6 테스트 설명하기

- 테스트에는 좋은 이름과 설명서가 있어야 함
- 주석을 추가하는 방법도 있지만 JUnit5를 활용하는 편이 훨씬 나음

```java
@DisplayName("Expect 44% after filling 221 in an empty 501 tank")
```

- 메서드를 바꾸지 않고도 훌륭한 테스트 설명을 추가할 수 있음
- 공백과 %, >과 같은 기호를 활용해 매우 표현적이고 간결한 테스트 설명을 작성할 수 있음
- 테스트를 비활성하는 이유도 꼭 설명해야 함

```java
@Disabled("We don't have small tanks anymore! TODO: Adapt for big tanks") 
```

- 이렇게 하면 얻는 이점
    
    1) 비활성화를 하며 나중에 테스트가 어떻게 변할지 생각할 수 있음
    
    2) 향후 개발자가 다시 활성화하려고 할 때 필요한 정보 제공


## 6.7 독립형 테스트 사용하기

- @BeforeEach와 @BeforeAll은 코드 중복이 없도록 도와주지만 설정 메서드로 인해 테스트를 이해하기 어려워짐
    - 클래스 전체를 맨 위부터 아래까지 읽는다면 매 단일 테스트 메서드마다 설정 메서드의 역할을 다시 떠올려야 함
    - 여러 설정 메서드가 클래스 계층 구조에 걸쳐 퍼져 있고 테스트가 많다면 이해하기 어려울 것임
- 해결방법은 테스트와 설정 코드를 더 분명히 연관짓는 것임
    - given, when, then 부분을 하나의 테스트 메서드 안에서 바로 연결해야 독립적인 테스트가 됨

## 6.8 테스트 매개변수화

- 여러 다양한 입력 매개변수로 테스트해야할 때 매개변수를 열거하면 테스트가 복잡해짐
- JUnit에는 이러한 상황에 맞는 특수한 어서션이 있는데, @ParameterizedTest와 @ValueSource 표기로 테스트를 매개변수화하는 것임 → 실제 테스트코드를 분리시킬 수 있음

## 6.9 경계 케이스 다루기

- 경우의 수를 모두 테스트하지 말고 가장 틀리기 쉬운 일반적인 실행 경로와 설정을 다루어야 함 (달리 말해, 경계 케이스를 다루어야 함)
- 일반적이지 않은 문자열이 들어올 경우 다음과 같은 입력들이 시스템 내 버그를 더 잘 찾음
    - null
    - “”
    - “ “
    - a\int~~(영문자가 아닌 특수문자를 포함하는 String)
