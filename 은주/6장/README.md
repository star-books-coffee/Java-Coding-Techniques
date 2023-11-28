## 6장. 올바르게 드러내기
### 6.1. Given - When - Then 으로 테스트 구조화

- `given` : 실제 테스트를 준비하는 단계,  테스트하려는 기능을 실행하기 위한 전제조건을 모두 포함
- `when` : 실제로 테스트하려는 연산을 수행
- `then` : when 에서 수행한 결과가 실제 기대한 결과인지 명확히 드러냄 (assertion)
- **given - when - then 을 주석으로 넣어 테스트코드 구조를 명확히하라**

### 6.2. 의미 있는 어서션 사용하기

```java
class CruiseControlTest {

    @Test
    void setPlanetarySpeedIs7667() {
        CruiseControl cruiseControl = new CruiseControl();

        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);

        Assertions.**assertTrue**(7667 == cruiseControl.getTargetSpeedKmh());
    }
}
```

- 이 테스트는 기능상 올바르게 동작하지만, 테스트 실패 시 java.lang.AssertionError 라는 스택추적을 받는데, 실패한 클래스/줄번호만 있고 메시지가 존재하지 않음
    - **어떤 assertion 이 실패했는지만 알 뿐, 왜 실패했는지를 모름**
    - assertion 이 두 값의 비교가 아니라 단지 어떤 boolean 값이 true 여야 하는데 fale 인 경우만 신경썼기 때문
    - assertTrue : 어떤 assertion 이든 결과는 조건을 만족하는 지 아닌지를 보여주는 불 값

```java
class CruiseControlTest {

    @Test
    void setPlanetarySpeedIs7667() {
        CruiseControl cruiseControl = new CruiseControl();

        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);

        Assertions.**assertEquals**(7667, cruiseControl.getTargetSpeedKmh());
    }
}
```

- 두 값이 같은지를 확인하는 assertEquals() 를 사용하면, 테스트가 왜 실패했는지도 알 수 있음 ex) expected: <7667> but was <1337>
    - assertTrue() 로도 자세한 메시지 전달할 수 있지만, 직접 만들어서 넣어야 함
- **더 나은 에러 메시지를 얻기 위해 검증하려는 테스트에 가장 적합한 assertion 을 선택하라**

### 6.3. 실제 값보다 기대 값을 먼저 보이기

```java
class CruiseControlTest {

    @Test
    void setPlanetarySpeedIs7667() {
        CruiseControl cruiseControl = new CruiseControl();

        cruiseControl.setPreset(SpeedPreset.PLANETARY_SPEED);

				// false
        Assertions.assertEquals(cruiseControl.getTargetSpeedKmh(), 7667);
				// true
        Assertions.**assertEquals**(7667, cruiseControl.getTargetSpeedKmh());
    }
}
```

- 해당 테스트를 실행하면 expected: <1337> but was <7667> 이라는 메시지를 출력하는데, 원래 7667 이 올바른 결과이므로 테스트 결과가 잘못 출력된 것임
- 자바나 JUnit 은 타입 검증을 지원하지 않기에, 인수를 올바른 순서로 작성해야 함
- 실패한 테스트의 메시지를 읽을 때 **최소한 그 메시지 자체는 무조건 옳다고 가정하므로 가정이 틀리면 안됨!**
- **assertEquals() 의 인수 순서에 주의를 기울여라**

### 6.4. 합당한 허용값 사용하기

```java
class OxygenTankTest {

    @Test
    void testNewTankIsEmpty() {
        OxygenTank tank = OxygenTank.withCapacity(100);
        Assertions.assertEquals(0, tank.getStatus());
    }

    @Test
    void testFilling() {
        OxygenTank tank = OxygenTank.withCapacity(100);

        tank.fill(5.8);
        tank.fill(5.6);

        Assertions.assertEquals(0.114, tank.getStatus());
    }
}
```

- 부동소수점 산술 연산은 근사화된 숫자일 뿐이므로, 부동소수점 연산을 테스트할 때는 소수점 자릿수를 명시해야 함

```java
class OxygenTankTest {
    static final double TOLERANCE = 0.00001;

    @Test
    void testNewTankIsEmpty() {
        OxygenTank tank = OxygenTank.withCapacity(100);

        Assertions.assertEquals(0, tank.getStatus(), TOLERANCE);
    }

    @Test
    void testFilling() {
        OxygenTank tank = OxygenTank.withCapacity(100);

        tank.fill(5.8);
        tank.fill(5.6);

        Assertions.assertEquals(0.114, tank.getStatus(), TOLERANCE);
    }
}
```

- assertEquals(double expected, double actual, double delta) assertion 은 delta 라는 허용값을 지원함
    - 소수점 둘째자리까지 일치해야 한다면, 0.001 을 허용값으로 사용하면 됨
- **assertEquals() 에 float 이나 double 을 사용할 때는 항상 자릿수를 알아야 하고 받아들일 수 있는 허용 수준을 명시하라**

### 6.5. 예외 처리는 JUnit 에게 맡기기

```java
class LogbookTest {

    @Test
    void readLogbook() {
        Logbook logbook = new Logbook();

        try {
            List<String> entries = logbook.readAllEntries();
            Assertions.assertEquals(13, entries.size());
        } catch (IOException e) {
            Assertions.fail(e.getMessage());
        }
    }

    @Test
    void readLogbookFail() {
        Logbook logbook = new Logbook();

        try {
            logbook.readAllEntries();
            Assertions.fail("read should fail");
        } catch (IOException ignored) {}
    }
}
```

- 첫번째 테스트 : 실패 시 메시지만 제공할 뿐, 전체 예외를 스택 추적으로 제공하지 않음
    - 원인 사슬이 깨짐
- 두번째 테스트 : 실제 예외가 일어나길 기대하는데, 코드만으로는 이유를 알 수 없음

```java
class LogbookTest {

    @Test
    void readLogbook() throws IOException {
        Logbook logbook = new Logbook();

        List<String> entries = logbook.readAllEntries();

        Assertions.assertEquals(13, entries.size());
    }

    @Test
    void readLogbookFail() {
        Logbook logbook = new Logbook();

        Executable when = () -> logbook.readAllEntries();

        Assertions.assertThrows(IOException.class, when);
    }
}
```

- 첫번째 테스트 : “어떤 예외도 발생하지 않는다” 라는 assertion 을 포함한 테스트
- 두번째 테스트 : assertThrows() 를 사용하여, 테스트에서 어떤 종류의 예외가 던져지길 바라는지 명시적으로 나타냄
- **예외가 생기길 바라는 메서드만 JUnit5 의 Executable 타입 형태로 assertThrows() 에 전달하라**

### 6.6. 테스트 설명하기

- 테스트코드를 잘설명하기 위해 **메서드명에 상황과 테스트 중인 메서드, 검증할 assertion 을 나타내라**
    - testFill2() → failOverfillTank()
- **@DisplayName 표기로, 간결한 테스트 설명을 작성하라**
    - `@DisplayName` (”Fail if fill level > tank capacity”)
- **테스트를 그냥 삭제하지 말고 @Disabled 표기에 왜 비활성화 하는지도 설명하라**
    - `@Disabled`(”[Why it’s disabled] TODO: [what’s the plan to enable again]”)

### 6.7. 독립형 테스트 사용하기

```java
class OxygenTankTest {
    OxygenTank tank;

    @BeforeEach
    void setUp() {
        tank = OxygenTank.withCapacity(10_000);
        tank.fill(5_000);
    }

    @Test
    void depressurizingEmptiesTank() {
        tank.depressurize();

        Assertions.assertTrue(tank.isEmpty());
    }

    @Test
    void completelyFillTankMustBeFull() {
        tank.fillUp();

        Assertions.assertTrue(tank.isFull());
    }
}
```

- @BeforeEach, @BeforeAll 표기는 테스트의 **given 부분에 필요한 공통 설정 코드를 추출하여 한번만 작성할 수 있게 해줌**
    - 하지만 설정메서드로 인해 매 단일 테스트 메서드마다 설정 메서드 역할을 떠올려야 함
    - 테스트가 더이상 `독립적이지 않음`

```java
class OxygenTankTest {
    static OxygenTank createHalfFilledTank() {
        OxygenTank tank = OxygenTank.withCapacity(10_000);
        tank.fill(5_000);
        return tank;
    }

    @Test
    void depressurizingEmptiesTank() {
        OxygenTank tank = **createHalfFilledTank**();

        tank.depressurize();

        Assertions.assertTrue(tank.isEmpty());
    }

    @Test
    void completelyFillTankMustBeFull() {
        OxygenTank tank = createHalfFilledTank();

        tank.fillUp();

        Assertions.assertTrue(tank.isFull());
    }
}
```

- given 부분을 @BeforeEach 설정 메서드로 분리하는 대신 각 테스트에 static 메서드 형태로 분리하여 연결시켰음
- @BeforeEach, @BeforeAll 표기가 만들어내는 암묵적 종속성은 코드를 읽기 힘들게 만드므로, 
**given 부분을 static 메서드 형태로 분리하여 테스트와 연결시켜, 테스트와 설정 코드를 분명하게 연결지어라**

### 6.8. 테스트 매개변수화

```java
class DistanceConversionTest {

    @Test
    void testConversionRoundTrip() {
        assertRoundTrip(1);
        assertRoundTrip(1_000);
        assertRoundTrip(9_999_999);
    }

    private void assertRoundTrip(int kilometers) {
        Distance expectedDistance = new Distance(
                DistanceUnit.KILOMETERS,
                kilometers
        );

        Distance actualDistance = expectedDistance
                .convertTo(DistanceUnit.MILES)
                .convertTo(DistanceUnit.KILOMETERS);

        Assertions.assertEquals(expectedDistance, actualDistance);
    }
}
```

- 메서드 하나를 같은 방법으로 테스트하지만, **여러 다양한 입력 매개변수로 테스트하고 싶을 때** 위와 같은 코드를 짜게 된다
- 하지만 하나의 테스트코드가 실패하게 되면, 그 뒤에 assertion 은 뛰어넘게 됨
- 매개변수 집합을 순회하는 for 루프를 테스트 메서드에 넣어도 문제는 고칠 수 없음

```java
class DistanceConversionTest {

    @ParameterizedTest(name = "#{index}: {0}km == {0}km->mi->km")
    @ValueSource(ints = {1, 1_000, 9_999_999})
    void testConversionRoundTrip(int kilometers) {
        Distance expectedDistance = new Distance(
                DistanceUnit.KILOMETERS,
                kilometers
        );

        Distance actualDistance = expectedDistance
                .convertTo(DistanceUnit.MILES)
                .convertTo(DistanceUnit.KILOMETERS);

        Assertions.assertEquals(expectedDistance, actualDistance);
    }
}
```

- 매개변수별로 각 테스트를 실행하고, 테스트당 assertion 을 하나씩만 넣도록 수정한다
- **@ParameterizedTest 와 @ValueSource 표기로 테스트를 매개변수화 시켜라**
    - 매개변수와 테스트코드를 분리시킬 수 있음
    - @ParameterizedTest 의 name 속성으로, “테스트 설명하기” 적용 가능

### 6.9. 경계 케이스 다루기

- **테스트 코드는 가능한 경우의 수를 모두 테스트하는 게 아니라, 가장 틀리기 쉬운 `경계 케이스` 를 다뤄라**
    - 매개변수의 **데이터 타입 경계** 정도는 최소한 테스트해라
    
    > int : 0, 1, -1, Integer.MAX_VALUE, Integer.MIN_VALUE, <br>
    double : 0, 1.0, -1.0, Double.MAX_VALUE, Double.MIN_VALUE,<br>
    Object[] : null, {}, {null}, {new Object(), null},<br>
    List<Object> : null, Collections.emptyList(), Collections.singletonList(null), Arrays.asList(new Object(), null)
    >
