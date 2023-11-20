## 3장. 슬기롭게 주석 사용하기
### 3.1. 지나치게 많은 주석 없애기

```java
class Inventory {
    // 필드 (우리는 하나만 가지고 있습니다)
    List<Supply> supplies = new ArrayList<>(); // 공급품 목록.

    // 메서드
    int countContaminatedSupplies() {
        // TODO: 필드가 이미 초기화되었는지 확인 (null이 아닌지)

        int contaminatedCounter = 0; // 오염된 공급품을 세는 카운터
        // 공급품이 없으면 오염이 없음
        for (Supply supply : supplies) { // FOR 루프 시작
            if (supply.isContaminated()) {
                contaminatedCounter++; // 카운터 증가!
            } // IF: 공급품이 오염되었는지 확인
        } // FOR 루프 종료

        // 오염된 공급품의 수를 반환합니다.
        return contaminatedCounter; // 주의해서 다루세요!
    }
} // Inventory 클래스 종료
```

- 대부분의 주석은 **코드가 전하는 내용을 반복할 뿐이니 불필요**하다

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    int countContaminatedSupplies() {
        if (supplies == null || supplies.isEmpty()) {
            // No supplies => no contamination
            return 0;
        }

        int contaminatedCounter = 0;
        for (Supply supply : supplies) {
            if (supply.isContaminated()) {
                contaminatedCounter++;
            }
        }

        return contaminatedCounter;
    }
}
```

- 코드 한 줄만 읽으면, 바로 알 수 있는 주석은 모두 제거하라 (코드를 바꾸어 설명하는 주석도 모두 제거)
- **코드만 보아서는 드러나지 않는 정보가 들어간 주석만 남겨라**
    - supplies == null 이거나 비었으면 0을 반환하는 온화한지속 (graceful continuation) 방법 선택
    - 온화한 지속 대신 예외를 발생시키는 방법을 선택할 수도 있기에 주석을 넣을 이유가 충분 !
    → 디자인 결정을 설명한 것
    - `graceful continuation` : 코드가 예외를 처리하고 프로그램이 비정상적으로 종료되지 않고 계속 실행되도록 하는 기술

### 3.2. 주석 처리된 코드 제거

- 주석 처리된 코드는 이해도를 떨어뜨리고, 혼란만 가중시키는 문제를 야기하므로 **다 지워라**

### 3.3. 주석을 상수로 대체

```java
enum SmallDistanceUnit {
    CENTIMETER,
    INCH;

    double getConversionRate(SmallDistanceUnit unit) {
        if (this == unit) {
            return 1; // identity conversion rate
        }

        if (this == CENTIMETER && unit == INCH) {
            return 0.393701; // one centimeter in inch
        } else {
            return 2.54; // one inch in centimeters
        }
    }
}
```

```java
enum SmallDistanceUnit {
    CENTIMETER,
    INCH;

    static final double INCH_IN_CENTIMETERS = 2.54;
    static final double CENTIMETER_IN_INCHES = 1 / INCH_IN_CENTIMETERS;
    static final int IDENTITY = 1;

    double getConversionRate(SmallDistanceUnit unit) {
        if (this == unit) {
            return IDENTITY;
        }

        if (this == CENTIMETER && unit == INCH) {
            return CENTIMETER_IN_INCHES;
        } else {
            return INCH_IN_CENTIMETERS;
        }
    }
}
```

- **주석을 코드에 임베딩해라 (주석 제거 후 상수명으로 통합)**
    - 상수는 **이름으로 의미를 드러내므로** 주석으로 설명할 필요가 없음
- 주석은 시간이 지나도 변하지 않을 위험성을 함께 내포하므로, 이름으로 코드를 설명하면 코드 변경 시 무시할 일이 거의 없다

### 3.4. 주석을 유틸리티 메서드로 대체

```java
class FuelSystem {

    List<Double> tanks = new ArrayList<>();

    int getAverageTankFillingPercent() {
        double sum = 0;
        for (double tankFilling : tanks) {
            sum += tankFilling;
        }
        double averageFuel = sum / tanks.size();
        // round to integer percent
        return Math.toIntExact(Math.round(averageFuel * 100));
    }
}
```

- 주석을 지우고, 명명된 변수를 생성하여 개선할 수 있다

```java
class FuelSystemAlternative {

    List<Double> tanks;

    int getAverageTankFillingPercent() {
        double sum = 0;
        for (double tankFilling : tanks) {
            sum += tankFilling;
        }
        double averageFuel = sum / tanks.size();
        int roundedToPercent = Math.toIntExact(Math.round(averageFuel * 100));
        return roundedToPercent;
    }
}
```

- 하지만 메서드에 추가한 변수가 불필요하므로, 유틸리티 메서드를 사용하여 수정할 것이다

```java
class FuelSystem {

    List<Double> tanks = new ArrayList<>();

    int getAverageTankFillingPercent() {
        double sum = 0;
        for (double tankFilling : tanks) {
            sum += tankFilling;
        }
        double averageFuel = sum / tanks.size();
        return roundToIntegerPercent(averageFuel);
    }

    static int roundToIntegerPercent(double value) {
        return Math.toIntExact(Math.round(value * 100));
    }
}
```

- roundToIntegerPercent() 라는 메서드를 추가하여, **코드가 무엇을 하는지 이름만으로 설명하라**
- 다른 메서드에서 새 메서드를 `재사용` 하여 코드를 `모듈화` 할 수 있게 된다
- 메서드의 계층 구조가 생겨 상위 계층 메서드의 이해도가 개선된다
    - getAverageTankFillingPercent() 는 하위 메서드 roundToIntegerPercent() 를 호출
    - 각 메서드는 **비슷한 추상화 정도**를 갖는 명명된 명령문의 나열이 됨
        - Q. getAverageTankFillingPercent()는 어떤 값들의 평균을 계산하고, 그 값을 정수 퍼센트로 반올림하는 일반적인 작업이고, roundToIntegerPercent() 메서드는 그 일을 수행하는 구체적인 단계 중 하나인데, 어떻게 이 두 메서드가 유사한 추상화 수준일까?
        - A. 유사한 추상화 수준 : 주로 상위 수준의 메서드와 하위 수준의 메서드 간의 연관성을 나타내며, 각 메서드가 서로 유사한 개념적 수준에서 작동한다는 의미
- 코드가 더 모듈화되고, 추상화 수준도 균형을 이룸

### 3.5. 구현 결정 설명하기

```java
class Inventory {
	
	private List<Supply> list = new ArrayList<>();
	
	void add(Supply supply) {
		list.add(supply);
		Collections.sort(list);
	}

	boolean isInStock(String name) {
		// 빠른 구현
		return Collections.binarySearch(list, new Supply(name)) != -1;
```

- 객관적으로 옳거나 그른 것이 없는 상황, 장단점이 모두 있는 상황에서는 주석이 필요
- 주석이 **선택의 근거를 타당하게 설명하고 있지 않음**
    - 왜 빠른가? 이 코드가 왜 빨라야 하는 가? binarySearch 는 정말 빠른가? 해법의 비용이나 트레이드오프는 무엇인가?

```java
class Inventory {
	// 리스트를 정렬된 채로 유지한다. isInStock()을 참고한다.
	private List<Supply> list = new ArrayList<>();
	
	void add(Supply supply) {
		list.add(supply);
		Collections.sort(list);
	}

	boolean isInStock(String name) {
		/*
		 * 재고가 남았는지 재고명으로 확인해야 한다면,            // 사용사례
		 * 재고가 천 개 이상일 때 심각한 성능 이슈에 직면한다.    // 우려사항
		 * 1초 안에 항목을 추출하게 위해                        // 품질
		 * 비록 재고를 정렬된 채로 유지해야 하지만               // 단점
	   * 이진 검색 알고리즘을 쓰기로 결정했다.                 // 해법
	   */
		return Collections.binarySearch(list, new Supply(name)) != -1;
```

- 중요한 결정이나 코드에서 까다로운 부분 설명시 아래의 템플릿을 사용하라
- **[사용 사례] 의 맥락에서
직면하는 [우려사항] 과 
우리가 선택한 [해법] 으로 
얻게 되는 [품질] 과
받아들여야 하는 [단점]**

### 3.6. 예제로 설명하기

```java
class Supply {

    /**
     * 코드는 공급품을 전 세계적으로 식별합니다.
     *
     * 엄격한 형식을 따릅니다. 'S'로 시작하여 다섯 자리의 재고 번호가 뒤따릅니다.
     * 그 다음에는 슬래시가 오며, 이는 국가 코드를 앞의 재고 번호와 구분합니다.
     * 이 국가 코드는 정확히 두 개의 대문자로 이루어져 있어야 하며,
     * 참여하는 국가 중 하나를 나타냅니다 (US, EU, RU, CN).
     * 그 뒤에는 점(.)이 오며, 소문자로 된 공급품의 실제 이름이 이어집니다.
     */
    static final Pattern CODE =
            Pattern.compile("^S\\d{5}\\\\(US|EU|RU|CN)\\.[a-z]+$");
}
```

- 복잡한 정규식을 설명하는 주석이 있어서 좋은 방법 같아 보임
- 하지만, 숙련된 개발자라면 **정규식 코드만으로 읽을 수 있는 내용을 그대로 반복하고 있다는 문제**가 있음

```java
class Supply {

    /**
     * 이 표현식은 공급품 코드를 전 세계적으로 식별합니다.
     *
     * 형식: "S<재고-번호>\\<국가-코드>.<이름>"
     *
     * 유효한 예제: "S12345\\US.pasta", "S08342\\CN.wrench",
     * "S88888\\EU.laptop", "S12233\\RU.brush"
     *
     * 무효한 예제:
     * "R12345\\RU.fuel"      (자원, 공급품이 아님)
     * "S1234\\US.light"      (다섯 자릿수 필요)
     * "S01234\\AI.coconut"   (잘못된 국가 코드. US, EU, RU, CN 중 하나 사용)
     * " S88888\\EU.laptop "  (뒷부분의 공백)
    */
    static final Pattern SUPPLY_CODE =
            Pattern.compile("^S\\d{5}\\\\(US|EU|RU|CN)\\.[a-z]+$");
}
```

- **Format 도 줄글 형태로 정리하는 것이 아니라, 반 자연어로 형식을 설명해라**
- **유효한 예제 + 유효하지 않은 예제를 제공해서 훨씬 이해하기 쉽게 바꿔라**

### 3.7. 패키지를 JavaDoc 으로 구조화하기

```java
/**
 * logistics 라는 이 패키지에는 물류(logistics)를 위한 클래스들이 포함되어 있습니다.
 * 이 패키지의 Inventory 클래스는 화물선(cargo ship)으로부터 입고를 받을 수 있으며,
 * 오염된 공급품을 처리할 수 있습니다.
 * 이 패키지의 클래스들:
 * - Inventory
 * - Supply
 * - Hull
 * - CargoShip
 * - SupplyCrate
 *
 * @author A. Lien, H. Uman
 * @version 1.8
 * @since 1.7
 */
package logistics;
```

- JavaDoc : 자바 API 가 제공하는 문서화 기능으로, **코드에서 public 인 요소를 설명**하는 데 사용됨
- **간략한 요약문**으로 시작, **상세한 설명**이 뒤를 이음, 패키지 내 모든 클래스 목록 등장
1. 패키지 내 클래스를 어떻게 사용하는지에 대한 정보가 없어 추상적
2. 전체 클래스 목록은 불필요
3. 마지막 표기 : 버전 관리 도구에 들어있는 중복 정보

```java
/**
 * 공급품(inventory)을 관리하기 위한 클래스들입니다.
 *
 * <p>
 * 핵심 클래스는 {@link logistics.Inventory}로, 이 클래스를 통해
 * <ul>
 * <li> {@link logistics.CargoShip}로부터 입고할 수 있고,
 * <li> 오염된 {@link logistics.Supply}를 처리할 수 있으며,
 * <li> 이름으로 {@link logistics.Supply}를 검색할 수 있습니다.
 * </ul>
 *
 * <p>
 * 이 클래스들은 공급품을 내리고 즉시 오염된 공급품을 처리할 수 있습니다.
 * <pre>
 * Inventory inventory = new Inventory();
 * inventory.stockUp(cargoShip.unload());
 * inventory.disposeContaminatedSupplies();
 * inventory.getContaminatedSupplies().isEmpty(); // true
 * </pre>
 */
package logistics;
```

1. 패키지 내 클래스로 **무엇을 할 수 있는지 매우 짧은 요약** 제공하라
2. 패키지 내 **주요 클래스로 무엇을 할 수 있는 지** 설명하라
3. 버전 관리 도구에 들어있는 정보 중복이 아니라, **주요 사용 사례를 어떻게 구현하는지 보여주는 구체적인 예제** 제공하라

### 3.8. 클래스와 인터페이스를 JavaDoc 으로 구조화하기

```java
/**
 * 이 클래스는 화물선(cargo ship)을 나타냅니다.                         // 요약문
 * 이 화물선은 {@link Stack} 형태의 공급품을 내릴(unload) 수 있으며,     // 상세설명
 * {@link Queue} 형태의 공급품을 실을(load) 수 있습니다.
 * 그리고 남은 용량을 long 형태로 보여줄 수 있습니다.
 */
interface CargoShip {
    Stack<Supply> unload();
    Queue<Supply> load(Queue<Supply> supplies);
    int getRemainingCapacity();
}
```

- 요약과 상세 설명이 **수직적 분리 없이** 붙어 있음
- 요약문 - 단순히 인터페이스명 반복하기만 함
- 상세 설명 - 인터페이스 메서드 서명 되풀이함

```java
/**
 * 화물선은 자체의 수용량에 따라 공급품을 싣거나 내릴 수 있습니다.
 *
 * <p>
 * 공급품은 순차적으로 실리며, LIFO (후입선출) 순서로 내릴 수 있습니다.
 * 화물선은 수용량까지만 공급품을 저장할 수 있습니다. 수용량은 양수이며,
 * 음수일 수 없습니다.
 */
interface CargoShip {
    Stack<Supply> unload();
    Queue<Supply> load(Queue<Supply> supplies);
    int getRemainingCapacity();
}
```

- 요약문 - 인터페이스명을 똑같이 되풀이 하지 않으면서, 유용한 조언을 제공하라
- 상세 설명 - LIFO 와 같이 동작을 상세히 설명하며, 인터페이스 호출 시 capacity 에 대해 보장하는 조건도 명시함
- **짧고 간결하지만 조언을 제공하는 요약으로 시작하고, 요약과 상세 설명을 수직적으로 분리하라**

### 3.9. 메서드를 JavaDoc 으로 구조화하기

```java
interface CargoShip {
    
    Stack<Supply> unload();

    /**
		 * {@link Supply}를 싣습니다.
		 *
		 * @param supplies {@link Queue} 형식의 공급품
		 * @return 실리지 않은 {@link Supply} 형식의 공급품
		 */
    Queue<Supply> load(Queue<Supply> supplies);

    int getRemainingCapacity();
}
```

- 메서드는 객체의 동작을 표현하며, 메서드 호출 시 `상태 변경` 과 `부수 효과` 가 발생함
- 위 주석은 새로운 내용 없이 메서드 서명만 반복하고 있음
    - JavaDoc 주석이 있음에도 메서드가 어떻게 동작하는지 전혀 알 수 없음

```java
interface CargoShip {
    
    Stack<Supply> unload();

    /**
		 * 화물선에 공급품을 싣습니다.
		 *
		 * <p>
		 * 남은 용량만큼만 실을 수 있습니다.
		 *
		 * 예제:
		 * <pre>
		 * int capacity = cargoShip.getRemainingCapacity(); // 1
		 * Queue&lt;Supply> supplies = Arrays.asList(new Supply("Apple"));
		 * Queue&lt;Supply> spareSupplies = cargoShip.load(supplies);
		 * spareSupplies.isEmpty(); // true;
		 * cargoShip.getRemainingCapacity() == 0; // true
		 * </pre>
		 *
		 * @param supplies 실릴 공급품; null이 아니어야 함
		 * @return 용량 부족으로 인해 실을 수 없었던 공급품; 모두 실렸다면 비어 있음
		 * @throws NullPointerException 만약 supplies가 null이라면
		 * @see CargoShip#getRemainingCapacity() 용량 확인
		 * @see CargoShip#unload() 공급품 내리기
		 */
    Queue<Supply> load(Queue<Supply> supplies);

    int getRemainingCapacity();
}
```

- 입력과 내부 상태가 **특정 출력과 상태 변경을 어떻게 보장하는지 명시해라**
- **@see 표기로 다른 메서드도 참조해서 부수적인 정보를 알려라**
    - 메서드가 일으킨 효과 롤백 방법,
    - 메서드 호출로 야기된 상태 변화 관찰 방법을 설명함

### 3.10. 생성자를 JavaDoc 으로 구조화하기

```java
class Inventory {

    List<Supply> supplies;

		/**
     * 새로운 Inventory를 위한 생성자입니다.
     */
    Inventory() {
        this(new ArrayList<>());
    }

		/**
     * 새로운 Inventory를 위한 다른 생성자입니다.
     *
     * Inventory에 초기 공급품을 추가할 수 있습니다.
     */
    Inventory(Collection<Supply> initialSupplies) {
        this.supplies = new ArrayList<>(initialSupplies);
    }
}
```

- 생성자는 의미있고 알맞은 이름을 할당할 수 없는 특수한 메서드 유형이다
- 위 예제의 요약문 이러한 단점이 존재함
    - 새로운 정보를 전달하지 않을 뿐더러 쓸모가 없음
    - 2번째 생성자의 경우 supplies 를 어떻게 추가하는지에 대한 정보가 없음
    - 두 생성자의 관계 추론 불가능
- 생성자의 JavaDoc 주석은 **프로그래머가 생성자를 사용하는 데 필요한 모든 요소를 설명**해야 함

```java
class Inventory {

    List<Supply> supplies;

    /**
     * 빈 Inventory를 생성합니다.
     *
     * @see Inventory#Inventory(Collection) 초기 공급품을 포함하여 생성
     */
    Inventory() {
        this(new ArrayList<>());
    }

    /**
     * 초기 공급품을 포함한 Inventory를 생성합니다.
     *
     * @param initialSupplies 초기 공급품.
     *                        null이 아니어야 하며, 비어 있을 수 있습니다.
     * @throws NullPointerException 만약 initialSupplies가 null이면
     * @see Inventory#Inventory() 공급품 없이 생성
     */
    Inventory(Collection<Supply> initialSupplies) {
        this.supplies = new ArrayList<>(initialSupplies);
    }
}
```

- **사용하는 입력 매개변수에 대한 정보를 알려줘라**
- **@see 주석을 통해 두 생성자의 관계를 설명해라**
