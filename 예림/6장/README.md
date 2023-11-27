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
