## CHAPTER 12 Dealing with Inheritance

### 12.1 Pull Up Method (메서드 올리기)

반대 리팩터링: 메서드 내리기

``` javascript
class Employee { ... }

class Saleperson extends Employee {
    get name() { ... }
}

class Engineer extends Employee {
    get name() { ... }
}
```

=>

``` javascript
class Employee {
    get name() { ... }
}

class Saleperson extends Employee { ... }

class Engineer extends Employee { ... }
```

#### 배경

중복 코드 제거는 중요하지만, 중복된 코드 한 쪽의 변경이 다른 쪽에는 반영되지 않을 수 있다는 위험을 항상 수반한다.

메서드 올리기를 적용하기 가장 쉬운 상황은 메서드들의 본문 코드가 똑같을 때다. 하지만 테스트를 통해 여전히 잘 동작하는지 확인해야 한다.

#### 절차

- 똑같이 동작하는 메서드인지 면밀히 살펴본다.
- 메서드 안에서 호출하는 다른 메서드와 참조하는 필드들을 수퍼클래스에서도 호출하고 참조할 수 있는지 확인한다.
- 메서드 시그니처가 다르다면 함수 선언 바꾸기로 수퍼클래스에서 사용하고 싶은 형태로 통일한다.
- 수퍼클래스에 새로운 메서드를 생성하고, 대상 메서드의 코드를 복사해 넣는다.
- 정적 검사를 수행한다.
- 서브클래스 중 하나의 메서드를 제거한다.
- 테스트한다.
- 모든 서브클래스의 메서드가 없어질 떄까지 다른 서브클래스의 메서드를 하나씩 제거한다.

#### 예시

### 12.2 Pull Up Field (필드 올리기)

반대 리팩터링: 필드 내리기

``` java
class Employee { ... } // 자바 코드

class Saleperson extends Employee {
    private String name;
}

class Engineer extends Employee {
    private String name;
}
```

=>

``` javascript
class Employee {
    protected String name;
}

class Saleperson extends Employee { ... }
class Engineer extends Employee { ... }
```

#### 배경

서브클래스들이 독립적으로 개발되었거나 뒤늦게 하나의 계층구조로 리팩터링된 경우라면 일부 기능이 중복되어 있을 때가 있다. 분석 결과 필드들이 비슷한 방식으로 쓰인다고 판단되면 수퍼클래스로 끌어올린다.

이렇게 하면, 데이터 중복 선언을 없앨 수 있고 해당 필드를 사용하는 동작을 서브클래스에서 수퍼클래스로 옮길 수 있다.

#### 절차

- 후보 필드들을 사용하는 곳 모두가 그 필드들을 똑같은 방식으로 사용하는지 면밀히 살핀다.
- 필드들의 이름이 각기 다르다면 똑같은 이름으로 바꾼다
- 수퍼클래스에서 새로운 필드를 생성한다
- 서브클래스의 필드들을 제거한다.
- 테스트한다.

### 12.3 Pull Up Constructor Body (생성자 본문 올리기)

``` javascript
class Party { ... }

class Employee extends Party {
    constructor (name, id, monthlyCost) {
        super();
        this._id = id;
        this._name = name;
        this._monthlyCost = monthlyCost;
    }
}
```

=>

``` javascript
class Party {
    constructor(name) {
        this._name = name;
    }
}

class Employee extends Party {
    constructor (name, id, monthlyCost) {
        super(name);
        this._id = id;
        this._monthlyCost = monthlyCost;
    }
}
```

#### 배경

서브클래스들에서 기능이 같은 메서드들을 발견하면 함수 추출하기와 메서드 올리기를 차례로 적용하여 수퍼클래스로 옮긴다. 그런데 메서드가 생성자라면 호출 순서에 제약이 있기 때문에 조금 다른 식으로 접근해야 한다.

#### 절차

- 수퍼클래스에 생성자가 없다면 하나 정의한다. 서브클래스의 생성자들에서 이 생성자가 호출되는지 확인한다.
- 문장 슬라이드하기로 공통 문장 모두를 supert() 호출 직후로 옮긴다.
- 공통 코드를 수퍼클래스에 추가하고 서브클래스들에서는 제거한다. 생성자 매개변수 중 공통 코드에서 참조하는 값들을 모두 supert()로 건넨다.
- 테스트한다.
- 생성자 시작 부분으로 옮길 수 없는 공통 코드에는 함수 추출하기와 메서드 올리기를 차례로 적용한다.

#### 예시

#### 예시: 공통 코드가 나중에 올 때

생성자는 대부분 super()를 호출하여 공통 작업을 먼저 처리한 다음, 각 서브클래스에 필요한 추가 작업을 처리하는 식으로 동작한다. 가끔 공통 작업이 뒤에 오는 경우가 있는데 그 예시이다.

### 12.4 Push Down Method (메서드 내리기)

반대 리팩터링: 메서드 올리기

``` javascript
class Employee { 
    get quota { ... }
}

class Saleperson extends Employee { ... }
class Engineer extends Employee { ... }
```

=>

``` javascript
class Employee { ... }

class Saleperson extends Employee { ... }
class Engineer extends Employee {
    get quota { ... }
}
```

#### 배경

특정 서브클래스 하나에만 관련된 메서드를 수퍼클래스에서 제거하고 해당 서브 클래스에 추가하는 편이 깔끔하다.

#### 절차

- 대상 메서드를 모든 서브클래스에 복사한다
- 수퍼클래스에서 그 메서드를 제거한다
- 테스트한다
- 이 메서드를 사용하지 않는 모든 서브클래스에서 제거한다
- 테스트한다

### 12.5 Push Down Field (필드 내리기)

반대 리팩터링: 필드 올리기

``` java
class Employee { // 자바 코드
    private String quota;
}

class Saleperson extends Employee { ... }
class Engineer extends Employee { ... }
```

=>

``` java
class Employee { ... }

class Saleperson extends Employee { ... }
class Engineer extends Employee {
    private String quota;
}
```

#### 배경

서브클래스 하나에서만 사용하는 필드는 해당 서브클래스로 옮긴다.

#### 절차

- 대상 필드를 모든 서브클래스에 정의한다
- 수퍼클래스에서 그 필드를 제거한다
- 테스트한다
- 이 필드를 사용하지 않는 모든 서브클래스에서 제거한다
- 테스트한다

### 12.6 Replace Type COde with Subclasses (타입 코드를 서브클래스로 바꾸기)

- 반대 리팩터링: 서브클래스 제거하기
- 하위 리팩터링
  - 타입 코드를 상태/전략 패턴으로 바꾸기
  - 서브클래스 추출하기

``` javascript
function createEmployee(name, type) {
    return new Employee(name, type);
}
```

=>

``` javascript
function createEmployee(name, type) {
    switch (type) {
        case "engineer":       return new Engineer(name);
        case "salesperson":  return new Salesperson(name);
        case "manager":       return new Manager(name);
```

#### 배경

소프트웨어 시스템에서는 비슷한 대상들을 특정 특성에 따라 구분해야 할 떄가 자주 있다. 이런 일을 다루는 수단으로는 타입 코드(type code) 필드가 있다.

타입 코드만으로 특별히 불편한 상황은 없지만 그 이상의 무언가가 필요할 때가 있다. 서브클래스를 사용하면 이럼 조건부 로직을 다형성으로 바꾸기를 적용할 수 있다.

특정 타입에서만 의미가 있는 값을 사용하는 필드나 메서드가 있다면 서브클래스가 의미가 있다. 서브클래스 방식이 관계를 더 명확히 드러내 준다.

#### 절차

- 타입 코드 필드를 자가 캡슐화한다.
- 타입 코드 값 하나를 선택하여 그 값에 해당하는 서브클래스를 만든다. 타입 코드 게터 메서드를 오버라이드하여 해당 타입 코드의 리터럴 값을 반환하게 한다.
- 매개변수로 받은 타입 코드와 방금 만든 서브클래스를 매핑하는 선택 로직을 만든다
- 테스트한다
- 타입 코드 값 각각에 대해 서브클래스 생성과 선택 로직 추가를 반복한다 클래스 하나가 완성될 때마다 테스트한다
- 타입 코드 필드를 제거한다
- 테스트한다
- 타입 코드 접근자를 이용하는 메서드 모두에 메서드 내리기와 조건부 로직을 다형성으로 바꾸기를 적용한다

#### 예시: 직접 상속할 때

직원 코드

#### 예시: 간접 상속할 때

직원의 서브클래스가 '아르바이트'와 '정직원' 클래스가 이미 있어서 Employee를 직접 상속하는 방식으로는 타입 코드 문제에 대처할 수 없다고 가정할 때.

### 12.7 Remove Subclass (서브클래스 제거하기)

- 반대 리팩터링: 타입 코드를 서브 클래스로 바꾸기
- 1판에서의 이름: 하위클래스를 필드로 전환

``` javascript
class Person {
    get genderCode() { return "X"; }
}
class Male extends Person {
    get genderCode() { return "M"; }
}
class Female extends Person {
    get genderCode() { return "F"; }
}
```

=>

``` javascript
class Person {
    get genderCode() {
        return this._genderCode;
    }
}
```

#### 배경

서브클래싱은 다름을 프로그래밍하는 멋진 수단이지만, 한번도 활용되지 않을 수도 있고 필요로 하지 않는 방식으로 만들어진 기능에서만 쓰이기도 한다.

더 이상 쓰이지 않는 서브클래스는 수퍼클래스의 필드로 대체해 제거한다.

#### 절차

- 서브클래스와 생성자를 팩터리 함수로 바꾼다
- 서브클래스의 타입을 검사하는 코드가 있다면 그 검사 코드에 함수 추출하기와 함수 옮기기를 차례로 적용하여 수퍼클래스로 옮긴다. 하나 변경할 때마다 테스트한다
- 서브클래스 타입을 나타내는 필드를 수퍼클래스에서 만든다
- 서브클래스를 참조하는 메서드가 방금 만든 타입 필드를 이용하도록 수정한다
- 서브클래스를 지운다
- 테스트한다

#### 예시

### 12.8 Extract Superclass (수퍼클래스 추출하기)

``` javascript
class Department {
    get totalAnnualCost() { ... }
    get name() { ... }
    get headCount() { ... }
}

class Employee {
    get annualCost() { ... }
    get name() { ... }
    get id() { ... }
}
```

=>

``` javascript
class Party {
    get name() { ... }
    get annualCost() { ... }
}

class Department extends Party {
    get annualCost() { ... }
    get headCount() { ... }
}

class Employee extends Party {
    get annualCost() { ... }
    get id() { ... }
}
```

#### 배경

비슷한 일을 수행하는 두 클래스가 보이면 상속 매커니즘을 이용해서 비슷한 부분을 공통의 수퍼클래스로 옮겨 담을 수 있다.

상속은 프로그램이 성장하면서 수퍼클래스로 끌어올리고 싶은 공통 요소를 찾았을 때 수행하는 사례가 많다.

#### 절차

- 빈 수퍼클래스를 만든다. 원래의 클래스들이 새 클래스를 상속하도록 한다
- 테스트한다.
- 생성자 본문 올리기, 메서드 올리기, 필드 올리기를 차례로 적용하여 공통 원소를 수퍼클래스로 옮긴다
- 서브클래스에 남은 메서드들을 검토한다. 공통되는 부분이 있다면 함수로 추출한 다음 메서드 올리기를 적용한다.
- 원래 클래스들을 사용하는 코드를 검토하여 수퍼클래스의 인터페이스를 사용하게 될지 고민해본다.

#### 예시

연간 비용, 월간 비용이라는 개념, 이름이 공통 기능이라면 수퍼클래스로 추출한다.

### 12.9 Collapse Hierarchy (계층 합치기)

``` javascript
class Employee { ... }
class Salesperson extends Employee { ... }
```

=> 

``` javascript
class Employee { ... }
```

#### 배경

클래스 계층구조도 진화하면서 어떤 클래스와 그 부모가 너무 비슷해져서 독립적으로 존재해야 할 이유가 사라지는데, 그 때 그 둘을 하나로 합쳐야 할 시점이다.

#### 절차

- 두 클래스 중 제거할 것을 고른다
- 필드 올리기와 메서드 올리기 혹은 필드 내리기와 메서드 내리기를 적용하여 모든 요소를 하나의 클래스로 옮긴다
- 제거할 클래스를 참조하던 모든 코드가 남겨질 클래스를 참조하도록 고친다.
- 빈 클래스를 제거한다
- 테스트한다

### 12.10 Replace Subclass with Delegate (서브클래스를 위임으로 바꾸기)

``` javascript
class Order {
    get daysToShip() {
        return this._warehouse.daysToShip;
    }
}

class PriorityOrder extends Order {
    get daysToShip() {
        return this.prioirtyPlan.daysToShip;
    }
}
```

=>

``` javascript
class Order {
    get daysToShip() {
        return (this._prioirtyDelegate) ? this._priorityDelegate.daysToShip : this._warehouse.daysToShip;
    }
}

class PriorityOrder extends Order {
    get daysToShip() {
        return this._prioirtyDelegate.daysToShip;
    }
}
```

#### 배경

상속에는 단점이 있다. 무언가 달라져야 하는 이유가 여러 개여도 상속에서는 그중 단 하나의 이유만 선택해 기준을 삼아야 한다.

상속은 클래스들의 관계를 아주 긴밀하게 결합한다. 부모가 수정되면 자식들의 기능을 해진다.

위임(delegate)은 이상의 두 문제를 모두 해결해준다. 위임은 객체 사이의 일반적인 관계이므로 상호작용에 필요한 인터페이스를 명확히 정의할 수 있다.

서브클래스는 위임으로 변경이 가능하므로 처음에는 상속을 사용해도 문제가 생기면 위임으로 갈아탄다.

#### 절차

- 생성자를 호출하는 곳이 많다면 생성자를 팩터리 함수로 바꾼다.
- 위임으로 활용할 빈 클래스를 만든다. 이 클래스의 생성자는 서브클래스에 특화된 데이터를 전부 받아야 하며, 보통은 수퍼클래스를 가리키는 역참조도 필요하다.
- 위임을 저장할 필드를 수퍼클래스에 추가한다.
- 서브클래스 생성 코드를 수정하여 위임 인스턴스를 생성하고 위임 필드에 대입해 초기화한다.
- 서브클래스의 메서드 중 위임 클래스로 이동할 것을 고른다
- 함수 옮기기를 적용해 위임 클래스로 옮긴다. 원래 메서드에서 위임하는 코드는 지우지 않는다.
- 서브클래스 외부에도 원래 메서드를 호출하는 코드가 있다면 서브클래스의 위임 코드를 수퍼클래스로 옮긴다. 이때 위임이 존재하는지를 검사하는 보호 코드로 감싸야 한다. 호출하는 외부 코드가 없다면 원래 메서드는 죽은 코드가 되므로 제거한다.
- 테스트한다
- 서브클래스의 모든 메서드가 옮겨질 때까지 위 과정을 반복한다.
- 서브클래스들의 생성자를 호출하는 코드를 찾아서 수퍼클래스의 생성자를 사용하도록 수정한다.
- 테스트한다
- 서브클래스를 삭제한다.

#### 예시: 서브클래스가 하나일 때

공연 예약 클래스와 추가 비용을 다양하게 설정할 수 있는 프리미엄 예약용 서브클래스

#### 예시: 서브클래스가 여러 개일 때

### 12.11 Replace Superclass with Delegate (수퍼클래스를 위임으로 바꾸기)

1판에서의 이름: 상속을 위임으로 전환

``` javascript
class List { ... }
class Stack extends List { ... }
```

=>

``` javascript
class Stack {
    constructor() {
        this._storage = new List();
    }
}

class List { ... }
```

#### 배경

상속을 잘못 적용한 예로는 자바의 스택 클래스가 있다. 자바의 스택은 리스트를 상속하고 있는데, 데이터를 저장하고 조작하는 리스트의 기능을 재활용하겠다는 생각이 초래한 결과다. 리스트의 연산 중 스택에는 적용되지 않는 게 많음에도스택 인터페이스에 노출되는게 문제다. 스택에서 리스트 객체를 필드에 저장해 두고 필요한 기능만 위임한다면 더 좋은 방식이 된다.

이 역시 상속을 먼저 적용하고 나중에 문제가 생기면 수퍼클래스를 위임으로 바꾸는 방식으로 진행하면 좋다.

#### 절차

- 수퍼클래스 객체를 참조하는 필드를 서브클래스에 만든다. 이 필드를 위임 참조라 부르고 새로운 수퍼클래스 인스턴스로 초기화한다.
- 수퍼클래스의 동작 각각에 대응하는 전달 함수를 서브클래스에 만든다. 서로 관련된 함수끼리 그룹으로 묶어 진행하며, 그룹을 하나씩 만들 때마다 테스트한다.
- 수퍼클래스의 동작 모두가 전달 함수로 오버라이드되었다면 상속 관계를 끊는다.

#### 예시

