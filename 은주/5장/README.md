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
