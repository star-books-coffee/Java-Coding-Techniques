# 3장. 슬기롭게 주석 사용하기
## 3.1 지나치게 많은 주석 없애기

### 개선 전 코드

```java
class Inventory {
    // 필드(하나만 있음)
    List<Supply> supplies = new ArrayList<>(); // 제품 리스트

    // Methods
    int countContaminatedSupplies() {
        // TODO: 필드가 이미 초기화되었는지(널이 아닌지) 검증한다

        int contaminatedCounter = 0; // 카운터
        // No supplies => no contamination
        for (Supply supply : supplies) { // FOR 시작
            if (supply.isContaminated()) {
                contaminatedCounter++; // 카운터를 증가시킨다!
            } // 제품이 변질되었으면 IF 끝
        }// FOR 끝

        // 변질된 제품 개수를 반환한다.
        return contaminatedCounter; // 유의해 처리한다!
    }
} // Inventory 클래스 끝
```

- 대부분의 주석이 코드가 전하는 내용을 반복할 뿐이니 불필요

### 개선 후 코드

```java
class Inventory {

    List<Supply> supplies = new ArrayList<>();

    int countContaminatedSupplies() {
        if (supplies == null || supplies.isEmpty()) {
            // 재품이 없으면 오염도 없다는 뜻이다
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

- 코드 한 줄만 익으면 바로 알 수 있는 주석은 제거
- TODO 주석을 수정하는 대신 null과 isEmpty() 검증 추가
- 예외를 발생시키는 방법 대신 graceful continuation을 택했기 때문에 주석을 넣을 이유가 충분

## 3.2 주석 처리된 코드 제거

### 개선 전 코드

```java
class LaunchChecklist {

    List<String> checks = Arrays.asList(
            "Cabin Leak",
            // "Communication", // 휴스턴과 정말 통신하고 싶은가?
            "Engine",
            "Hull",
            // "Rover", // 내 생각에는 필요 없는데...
            "OxygenTank"
            //"Supplies"
    );

    Status prepareLaunch(Commander commander) {
        for (String check : checks) {
            boolean shouldAbortTakeoff = commander.isFailing(check);
            if (shouldAbortTakeoff) {
                //System.out.println("REASON FOR ABORT: " + item);
                return Status.ABORT_TAKE_OFF;
            }
        }
        return Status.READY_FOR_TAKE_OFF;
    }
}
```

- 주석 처리된 코드는 심각한 문제임. 혼란만 가중시킬 쓰레기를 코드에 끼얹는 셈

### 개선 후 코드

- 주석 처리된 코드는 항상 이해도를 떨어드리므로 삭제

## 3.3 주석을 상수로 대체

```java
static final double INCH_IN_CENTIMETERS = 2.54
```

- 주석을 사용하는 것보다 이렇게 코드로 설명하는 게 훨씬 낫다
- 주석을 상수나 변수, 필드, 메서드명으로 넣을 수 있다면 망설이지 말고 진행시켜

## 3.4 주석을 유틸리티 메서드로 대체

- 코드가 더 복잡해질 경우 어떻게 해야할까?
1. 명명된 변수 생성

```java
int roundedToPercent = Math.toIntExact(Math.round(averageFuel*100));
```

1. 유틸리티 메서드 사용

## 3.5 구현 결정 설명하기

### 개선 전 코드

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

- 프로젝트에서 어려운 결정을 내려야할 때 주석이 필요함
- 주석이 이 선택의 근거를 타당하게 설명하고 있지 않음

### 개선 후 코드

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
		 * 재고가 남았는지 재고명으로 확인해야 한다면,           [사용 사례]
		 * 재고가 천 개 이상일 때 심각한 성능 이슈에 직면한다.    [우려사항]
		 * 1초 안에 항목을 추출하게 위해                     [품질]
		 * 비록 재고를 정렬된 채로 유지해야 하지만              [단점]
	   * 이진 검색 알고리즘을 쓰기로 결정했다.                [해법]
	   */
		return Collections.binarySearch(list, new Supply(name)) != -1;
```

- 사용 사례와 우려사항, 해법, 그리고 지불해야할 트레이드 오프나 비용까지 명시
- 프로젝트 템플릿을 형성하는 건 도움이 됨

## 3.6 예제로 설명하기

### 개선 전 코드

```java
class Supply {

    /**
     * 아래 코드는 어디서든 재고를 식별한다.
     *
     *** S로 시작해 숫자 다섯자리 재고 번호가 나오고**
     * 뒤이어 앞의 재고 번호와 구분하기 위한 역 슬래시가 나오고
     * 국가 코드가 나오는 엄격한 형식을 따른다.
     * 국가 코드는 반드시 참여 국가인 (US, EU, RU, CN) 중
     * 하나를 뜻하는 대문자 두 개여야 한다
     * 이어서 마침표와 실제 재고명이 소문자로 나온다.
     */
    static final Pattern CODE =
            Pattern.compile("^S\\d{5}\\\\(US|EU|RU|CN)\\.[a-z]+$");
}
```

- 복잡한 만큼 더 쉽게 이해할 수 있도록 설명해야함
- 숙련된 개발자라면 정규식 코드만으로 읽을 수 있는 내용을 그대로 반복하고 있음
- 예제로 본을 보여야 함

### 개선 후 코드

```java
class Supply {

    /**
     * 아래 코드는 어디서든 재고를 식별한다.
     *
     * 형식: "S<inventory-number>\<COUNTRY-CODE>.<name>"
     *
     * 유효한 예: "S12345\US.pasta", "S08342\CN.wrench",
     * "S88888\EU.laptop", "S12233\RU.brush"
     *
     * 유효하지 않은 예:
     * "R12345\RU.fuel"      (재고가 아닌 자원)
     * "S1234\US.light"      (숫자가 다섯 개여야 함)
     * "S01234\AI.coconut"   (잘못된 국가 코드. US나 EU, RU, or CN 중 하나를 사용한다.)
     * " S88888\EU.laptop "  (마지막에 여백이 있음.)
    */
    static final Pattern SUPPLY_CODE =
            Pattern.compile("^S\\d{5}\\\\(US|EU|RU|CN)\\.[a-z]+$");
}
```

- 전보다 좀 길지만 더 구조적이고 정보도 더 많이 제공
- 예제를 단위 테스트로 추가해도 좋음

## 3.7 패키지를 JavaDoc으로 구조화하기

### 개선 전 코드

```java
/**
 *** logistics라는 이 패키지는 물류(logistics)를 위한 클래스를 포함한다.**
 * 이 패키지의 inventory 클래스는 화물선에 제품을 선적하고,
 * 변질된 클래스는 모두 버릴 수 있다.
 * 이 패키지의 클래스:
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

- JavaDoc은 자바 API가 제공하는 문서화 기능임
- 그러나 기본적으로 요약문은 불필요함
- 패키지 내 클래스를 어떻게 사용하는지 나와있지 않고, 전체 클래스 목록은 필요하지도 않음
- 이미 버전 관리 도구에 들어있는 중복 정보임

### 개선 후 코드

```java
/**
 * 제품 재고를 관리하는 클래스
 *
 * <p>
 * 주요 클래스는 {@link logistics.Inventory}로서 아래 기능을 수행한다.
 * <ul>
 * <li> {@link logistics.CargoShip}으로 선적하고,
 * <li> 변질된 {@link logistics.Supply}를 모두 버리고,
 * <li> 이름으로 어떤 {@link logistics.Supply}든 찾는다.
 * </ul>
 *
 *** <p>**
 * 이 클래스는 제품을 내리고 변질된 제품은 즉시 모두 버릴 수 있게 해준다.
 * <pre>
 * Inventory inventory = new Inventory();
 * inventory.stockUp(cargoShip.unload());
 * inventory.disposeContaminatedSupplies();
 * inventory.getContaminatedSupplies().isEmpty(); // true
 * </pre>
 */
package logistics;
```

- 패키지 내 클래스로 무엇을 할 수 있는지 매우 짧은 요약을 제공
- 패키지 내 주요 클래스로 무엇을 할 수 있는지 설명
- `@link`표기를 통해 간단히 클래스를 클릭해 바로 이동 가능
- `@author` 대신 주요 사용 사례(use case)를 어떻게 구현하는지 보여주는 구체적인 예제 제공

## 3.8 클래스와 인터페이스를 JavaDoc으로 구조화하기

### 개선 전 코드

```java
/**
 *** 이 클래스는 화물선을 나타낸다.**
 * 제품의 {@link Stack}를 내릴 수 있고 제품의 {@link Queue}을 실을 수 있으며
 * long 타입으로 remaingCapacity를 보여줄 수 있다.
 */
interface CargoShip {
    Stack<Supply> unload();
    Queue<Supply> load(Queue<Supply> supplies);
    int getRemainingCapacity();
}
```

- 모든 퍼블릭 클래스나 인터페이스는 JavaDoc으로 설명해야 함. 이것은 대부분의 자바 프로젝트에 적용되는 규칙임
- 위 예제의 문제점 :

### 개선 후 코드

```java
/**
 *** 화물선은 용량에 따라 제품을 싣고 내릴 수 있다.**
 * 
 * <p>
 * 제품은 순차적으로 선적되고 LIFO(last-in-first-out) 순으로 내려진다.
 * 화물선은 용량만큼만 제품을 저장할 수 있다.
 * 용량은 절대 음수가 아니다.
*/
interface CargoShip {
	Stack<Supply> unload;
	Queue<Supply> load(Queue<Supply> supplies);
	int getRemainingCapacity();
}
```

- 개선 된 사항 :
- JavaDocs 주석은 이렇게 작성하라

## 3.9 메서드를 JavaDoc으로 구조화하기

### 개선 전 코드

```java
interface CargoShip {
    
    Stack<Supply> unload();

    /**
     * {@link Supply}를 싣는다.
     *
     * @param {@link Queue} 타입의 제품 제공
     * @return {@link Queue} 타입의 적재되지 않은 제품
     */
    Queue<Supply> load(Queue<Supply> supplies);

    int getRemainingCapacity();
}
```

- 메서드는 객체의 동작을 표현함
- 메서드를 호출하면 상태 변경과 부수 효과가 발생함 → JavaDoc 설명이 중요
- 위 예제는 새로운 내용 없이 메서드 서명만 반복할 뿐임

### 개선 후 코드

```java
interface CargoShip {
    
    Stack<Supply> unload();

    /**
     * 제품을 화물선에 싣는다.
     *
     * <p>
     * 남은 용량만큼만 제품을 싣게 해준다.
     *
     * 예:
     * <pre>
     * int capacity = cargoShip.getRemainingCapacity(); // 1
     * Queue&lt;Supply> supplies = Arrays.asList(new Supply("Apple"));
     * Queue&lt;Supply> spareSupplies = cargoShip.load(supplies);
     * spareSupplies.isEmpty(); // true;
     * cargoShip.getRemainingCapacity() == 0; // true
     * </pre>
     *
     * @param 적재할 상품; 널이면 안된다
     * @return 용량이 작아 실을 수 없었던 제품; 모두 실었다면 empty
     * @throws 제품이 널이면 NullPointerException
     * @see CargoShip#getRemainingCapacity() 용량을 확인하는 함수
     * @see CargoShip#unload() 제품을 내리는 함수
     */
    Queue<Supply> load(Queue<Supply> supplies);

    int getRemainingCapacity();
}
```

- 입력과 내부 상태가 특정 출력과 상태 변경을 어떻게 보장하는지 명시
- null처럼 유효하지 않은 입력까지도 @param 설명에 명시
- @see 표기로 다른 메서드도 참조함

## 3.10 생성자를 JavaDoc으로 구조화하기

### 개선 전 코드

```java
class Inventory {

    List<Supply> supplies;

    /**
     * 새 Inventory의 생성자
     */
    Inventory() {
        this(new ArrayList<>());
    }

    /**
     * 새 Inventory의 또 다른 생성자
     *
     * 제품을 Inventory에 추가할 수 있는 생성자
     */
    Inventory(Collection<Supply> initialSupplies) {
        this.supplies = new ArrayList<>(initialSupplies);
    }
}
```

- 자바에는 의미 있고 알맞은 이름을 할당할 수 없는 특수한 메서드 유형이 하나 있음 → 생성자
- 생성자의 JavaDoc 주석은 프로그래머가 생성자를 사용하는 데 필요한 모든 요소를 설명해야 함

### 개선 후 코드

```java
class Inventory {

    List<Supply> supplies;

    /**
     * 빈 재고를 생성한다.
     *
     * @see Inventory#Inventory(Collection) 초기 제품을 초기화하는 함수
     */
    Inventory() {
        this(new ArrayList<>());
    }

    /**
     * 제품을 처음으로 선적한 재고를 생성한다.
     *
     * @param initialSupplies 제품을 초기화한다.
     *                        널이면 안 되고 빌 수 있다.
     * @throws NullPointerException initialSupplies이 널일 때
     * @see Inventory#Inventory() 제품없이 초기화하는 함수
     */
    Inventory(Collection<Supply> initialSupplies) {
        this.supplies = new ArrayList<>(initialSupplies);
    }
}
```

- 원하는대로 동작하려면 어떤 전제 조건을 충족해야 하는지 설명해야 함
- 위 예제에서는 생성자 종료 후 객체 상태 정보(사후 정보)를 보여줌 - 재고는 “빈” 상태가 되거나 “초기 제품”으로 채워짐
- 알려주지 않았다면 몰랐을 대안을 보여줌