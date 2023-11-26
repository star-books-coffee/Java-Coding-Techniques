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
