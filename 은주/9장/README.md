## 9장. 실전 준비
### 9.1. 정적 코드 분석 도구

- 코드를 분석하고, 잠재적 버그나 코드 스멜 (어떤 문제가 있음을 암시하는 코드) 를 찾는 도구
    - 컴파일 오류 + 기능적 오류를 넘어서는 특정 유형 문제를 코드에서 자동 감지/수정을 도와줌
- SpotBugs
    - 파일 또는 IDE 에 문제가 발생할 만한 코드조각을 알려줌 / 복사본+수정본 제공 X
    - 경고 받을 때마다 문제인지 아닌지 확인 필요
- Checkstyle / PMD
    - 특정 코드 방식을 고집할 때 유용함
- Error Prone
    - 자바 컴파일러 개선, 고급 타임 검증을 수행하여 수많은 이슈에 대한 수정본 제안
- Code Inspection
    - 자동으로 코드 내 이슈 감지하는 기능으로, 클릭 한번으로 코드 리팩토링 가능
- **컴파일러나 테스트로 찾지 못하는 코드 내 문제를 발견하는데 유용하므로 정적 코드 분석 도구를 활용하라**

### 9.2. 팀 내 자바 포맷 통일

- 계속된 재서식화, 버그를 피하기 위해 팀원 모두가 하나의 일관된 서식화 방식에 동의해야 한다.
- **실용적인 기본 값으로 자바 코드 규칙을 업데이트 한 버전인, 업계 표준 구글 자바 스타일 가이드를 사용하자**

### 9.3. 빌드 자동화

- 자바 개발자들은 다양한 IDE 를 사용해서 코드를 작성하는데, 하나의 IDE 를 사용할 수 없는 경우가 있다
- **모든 시스템에서 같은 방식으로 동작하고, 개발자 장비와도 독립적인 빌드 도구나 언어로 빌드를 자동화하라**
    - ex) gradle, maven
- 빌드 자동화를 위해 빌드 파일 작성 후 프로젝트에 넣어야 함
    - 종속성, 외부 라이브러리, 성공 빌드를 위해 무엇을 해야 하는지
- 빌드 도구 : 파일 해석 → 필요 외부 라이브러리 다운로드 → 모든 테스트 실행 → 테스트 성공 시 실행파일 빌드

### 9.4. 지속적 통합

- 로컬에서 빌드 도구로 언제든 테스트 실행 후 빌드가 가능해짐
- **버전 제어 시스템 커밋 → 지속적 통합 서버가 코드를 가져와 전체 테스트 실행 → 완벽히 통합된 실행 파일 빌드**
    - ex) Jenkins, SonarQube, Travis-CI

### 9.5. 생산 준비와 납품

- **로그, 수치, 대시보드, 알람 형태로 감시를 구현하고, 로그가 모일 중앙 위치 설정하여 로그 검색 및 분석이 가능하도록 해라**
    - ex) Open Source Elastic Stack

### 9.6. 콘솔 출력 대신 로깅

- **log4j 를 사용하여 콘솔 출력이 아니라 로그를 찍어서 추적하기 쉽도록 하고, 로그 수준으로 명령문의 심각도를 표현하라**

### 9.7. 다중 스레드 코드 최소화 및 독립

- 성능 벤치마크에서 심각한 결과가 나오지 않는 이상, 때이른 최적화와 다중 스레드 코딩은 위험하다 → 성능측정 후 너무 느릴 때만 다중 스레드를 활용하라
- **코드 기반에서 최소한 패키지에만 제한시키고, 가능하면 다중스레드 코드를 독립적으로 분리시키고 완벽히 설명해라**
    - 가변 데이터는 race condition (여러 프로세스가 공유 자원 동시 접근 시, 실행순서에 따라 결과값이 달라질 수 있는 현상) 이나 lost update (2개 이상의 트랜잭션이 같은 자원을 공유하며 갱신할 때 갱신 결과 일부가 사라지는 현상) 와 같은 동시 실행 버그 일으키기 쉬움
- **코드 내 가변 데이터에 동시 접근이 어떻게 이루어지는지, 데이터를 동시 실행 버그로부터 어떻게 보호하는지 설명하라**
    - JCIP 표기 사용
        - [JCIP annotation](http://jcip.net/annotations/doc/net/jcip/annotations/package-summary.html)은 'Java concurrent in practice' 책에서 제안된 스레드 안정성을 표시하는 기법으로, 4개의 종류의 애노테이션을 제공함
        - `@GuardedBy` : 해당 객체가 어떤 Lock으로 보호되고 있는지 표시. 필드에 메소드에 사용 가능
        - `@Immutable` : 불변객체
        - `@NotThreadSafe` : 스레드 안전하지 않음
        - `@ThreadSafe` : 스레드 안전함
        
        ```java
        @NotThreadSafe
        public class HttpGet extends HttpRequestBase {
        
        @ThreadSafe
        public class DefaultHttpClient extends AbstractHttpClient {
        
        @ThreadSafe
        public class SingleClientConnManager implements ClientConnectionManager {
        ```
        

### 9.8. 고급 동시 실행 추상화 사용하기

- `동기화 프리미티브` : volatile, synchronized 와 같은 키워드로 코드 내 임계영역 표시할 때 사용
    - 하지만 잘못사용하기 쉽고, fairness 와 같은 다중 스레드 요구사항 만족 시키기 까다로움
    - 따라서 Semaphore, CountDownLatch 와 같은 동기화 클래스 + AtomicInteger, ConcurrentHashMap, BlockingQueue 와 같은 자료구조 제공
        
        → 이러한 클래스를 올바르게 사용하려면 **자바 메모리 모델과 모델 내 상태변경 간 전후관계가 어떻게 동작하는지** 완벽히 이해해야 함
        

### 9.9. 프로그램 속도 향상

```java
class Inventory {
    List<Supply> supplies;

    long countDifferentKinds() {
        return supplies.stream()
                       .sequential() // 생략 가능
                       .filter(Supply::isUncontaminated)
                       .map(Supply::getName)
                       .distinct()
                       .count();
    }
}
```

- sequential() 호출 시, 전체 스트림이 원소를 순차적으로 처리함 (stream 의 기본 동작이므로 명시 호출할 필요는 없음)
- `부수 효과가 없는` 순차 스트림은 코어 여러 개에 분산해 병렬로 작업 실행 가능

```java
class Inventory {
    List<Supply> supplies;

    long countDifferentKinds() {
        return supplies.stream()
                       .parallel()
                       .filter(Supply::isUncontaminated)
                       .map(Supply::getName)
                       .distinct()
                       .count();
    }
}
```

- sequential() → parallel() 로 바꾸기만 하면 됨
- streamParallel() 을 사용해서 병렬 스트림을 생성할 수 있지만, 많은 메서드들이 제공되지 않아서 parallel() 을 더 다용도로 사용됨
- **부수효과가 없는 스트림일 때, 매 처리 단계가 서로 독립적일 때만 동작함**
- **성능 향상을 꾀할 때는, 항상 의도한 효과만큼 향상되는지 확인해야 한다.**
    - sort(), forEachOrdered() 처럼 스트림을 다시 동기화하는 연산을 사용하면 오버헤드가 매우 커서 sequential() 연산이 더 빠를 수도 있음

### 9.10. 틀린 가정 알기

- **형식에 대해 상세히 알지 못하면 코드에서 너무 많이 가정하지 말고, 그 작업을 클래스의 사용자에게 넘겨서 처리하라**
    - ex) 이메일 주소, 우편번호, csv 파일, 표준시간대 처리 (다양한 방식이 존재하므로)
- 현실 세계에 맞도록 코드를 준비하려면 순전히 가정만으로 해서는 모든 케이스를 커버하지 못할 수 있음
