상속은 발등에 불이 떨어져서야 비로소 잘못 사용했을음 알아차기는 경우가 많다. 

# 12.1 메서드 올리기

```jsx
class Employee { ... }

class Salesperson extends Employee {
	get name() { ... }
}

class engineer extends Employee {
	get name() { ... }
}
```

```jsx
class Employee {
	get name() {...}
}

class Salesperson extends Employee {...}

class engineer extends Employee {...}
```
## 다른 예제
```js
class Party {}

class Department extends Party {
  get totalAnnualCost() {
    return this.monthlyCost * 12;
  }
}
class Employee extends Party {
  get annualCost() {
    return this.monthlyCost * 12;
  }
}
```
```js
class Party {
	get annualCost() {
    return this.monthlyCost * 12;
  }	
}

class Department extends Party {}
class Employee extends Party {}
```

## 배경

- 메서드 올리기를 적용하기 가장 쉬운 상황은 메서드들의 본문 코드가 똑같을 때다.
- 두 메서드의 전체 흐름은 비슷하지만, 세부 내용이 다르다면 템플릿 메서드 만들기를 고려하자.

## 절차

1. 똑같이 동작하는 메서드인지 면밀히 살펴본다.
→ 실질적으로 하는 일을 같지만, 코드가 다르다면 본문 코드가 똑같아질 때까지 리팩터링한다.
2. 메서드 안에서 호출하는 다른 메서드와 참조하는 필드들을 슈퍼클래스에서도 호출하고 참조할 수 있는 확인 한다. 
3. 메서드 시그니처가 다르다면 함수 선언 바꾸기로 슈퍼클래스에서 사용하고 싶은 형태로 통일한다.
4. 슈퍼클래스에 새로운 메서드를 생성하고, 대상 메서드의 코드를 복사해 넣는다. 
5. 정적 검사를 수행한다. 
6. 서브 클래스 중 하나의 메서드를 제거한다. 
7. 테스트한다.
8. 모든 서브클래스의 메서드가 없어질 때까지 다른 서브 클래스의 메서드를 하나씩 제거한다. 

# 12.2 필드 올리기
- 상속을 사용할 때는 어떤 클래스가 필요한지 공통적인 속성을 생각해서 부모클래스를 정의한다.
```java
class Employee { ... }

class Salesperson extends Employee {
	private String name
}

class engineer extends Employee {
	private String name
}
```

```java
class Employee {
	protected String name
}

class Salesperson extends Employee {...}

class engineer extends Employee {...}
```

## 배경
- 분석 결과 필드들이 비슷한 방식으로 쓰인다고 판단되면 슈퍼클래스로 끌어올리자.
- 두 가지 중복을 줄일 수 있다.
    - 데이터 중복 선언을 없앨 수 있다.
    - 해당 필드를 사용하는 동작을 서브 클래스에서 슈퍼클래스로 옮길 수 있다.

## 절차

1. 후보 필드들을 사용하는 곳 모두가 그 필드들을 똑같은 방식으로 사용하는지 면밀히 살핀다.
2. 필드들의 이름이 각기 다르다면 똑같은 이름으로 바꾼다(필드 이름 바꾸기)
3. 슈퍼클래스에 새로운 필드를 생성한다.
→ 서브 클래스에서 이 필드에 접근할 수 있어야 한다(대부분 언어에서는 protected로 선언하면 된다).
4. 서브 클래스의 필드들을 제거한다.
5. 테스트한다.

# 12.3 생성자 본문 올리기

```jsx
class Party {...}

class Employee extends Party {
	constructor(name, id, monthlyCost){
		super();
		this._id = id;
		this._name = name;
		this._monthlyCost = monthlyCost;
	}
}
```

```jsx
class Party {
	constructor(name) {
		this._name = name;
	}
}

class Employee extends Party {
	constructor(name, id, monthlyCost){
		super(name);
		this._id = id;
		this._monthlyCost = monthlyCost;
	}
}
```

## 배경

- 서브 클래스들에서 기능이 같은 메서드들을 발견하면 함수 추출하기와 메서드 올리기를 차례로 적용하여 말끔히 슈퍼클래스로 옮기곤 한다.

## 절차

1. 슈퍼클래스에 생성자가 없다면 하나 정의한다. 서브 클래스의 생성자들에서 이 생성자가 호출되는지 확인한다.
2. 문장 슬라이드 하기로 공통 문장을 `super()` 호출 직후로 옮긴다.
3. 공통 코드를 슈퍼클래스에 추가하고 서브클래스들에는 제거한다. 생성자 매개변수 중 공통 코드에서 참조하는 값들을 모두 `super()`로 건넨다.
4. 테스트한다.
5. 생성자 시작 부분으로 옮길 수 없는 공통 코드에는 함수 추출하기와 메서드 올리기를 차례로 적용한다.

# 12.4 메서드 내리기

```jsx
class Employee {
	get quota {...}
}

class Engineer extends Employee {...}
class Salesperson extends Employee {...}
```

```jsx
class Employee {...}

class Engineer extends Employee {...}
class Salesperson extends Employee {
	get quota {...}
}
```

## 배경

- 특정 서브 클래스 하나(혹은 소수)와만 관련된 메서드는 슈퍼클래스에서 제거하고 해당 서브 클래스(들)에 추가하는 편이 좋다.

## 절차

1. 대상 메서드를 모든 서브 클래스에 복사한다.
2. 슈퍼클래스에서 그 메서드를 제거한다.
3. 테스트한다.
4. 이 메서드를 사용하지 않는 모든 서브 클래스에서 제거한다.
5. 테스트한다.

# 12.5 필드 내리기

```java
class Employee {
	private String quota
}

class Engineer extends Employee {...}
class Salesperson extends Employee {...}
```

```java
class Employee {...}

class Engineer extends Employee {...}
class Salesperson extends Employee {
	protected String quota;
}
```

## 배경

- 서브클래스 하나(혹은 소수)에서만 사용하는 필드는 해당 서브클래스(들)로 옮긴다.

## 절차

1. 대상 필드를 모든 서브 클래스에 정의한다.
2. 슈퍼클래스에서 그 필드를 제거한다.
3. 테스트한다.
4. 이 필드를 사용하지 않는 모든 서브 클래스에서 제거한다.
5. 테스트한다.

# 12.6 타입 코드를 서브클래스로 바꾸기

```jsx
function createEmployee(name, type) {
	return new Employee(name, type);
}
```
```jsx
function createEmployee(name, type) {
	switch (type) {
		case "engineer": return new Engineer(name);
		case "salesperson": return new Salesperson(name);
		case "manager": return new Manager(name);
	}
}
```
## 다른 예제
- 생성자에서 에러를 던지는건 매우 안 좋다.
```js
class Employee {
  #name;
  #type;
  constructor(name, type) {
    this.validateType(type);
    this.#name = name;
    this.#type = type;
  }

  validateType(arg) {
    if (!['engineer', 'manager', 'salesperson'].includes(arg)) {
      throw new Error(`${arg}라는 직원 유형은 없습니다.`);
    }
  }

  get type() {
    return this.#type;
  }

  toString() {
    return `${this.#name} (${this.type})`;
  }
}

const yongho = new Employee('용호', 'engineer');
const bob = new Employee('밥', 'manager');

```
```js
class Employee {
  #name;
  constructor(name, type) {
    this.#name = name;
  }

  get type() {
    return 'employee';
  }

  toString() {
    return `${this.#name} (${this.type})`;
  }
	
	static createEmployee(name, type){
		switch(type){
			case 'engineer':
					return new Engineer(name, type);
			case 'salesperson':
				return new Salesperson(name, type);
			case 'manager':
				return new Manager(name, type);
			default:
				throw new Error(`${type}라는 직원 유형은 없습니다.`);
		}
	}
}

class Engineer extends Employee{
	get type(){
		return 'Engineer';
	}
}

class Manager extends Employee{
	get type(){
		return 'Manager';
	}
}

const yongho = new Engineer('용호');
const bob = new Manager('밥');
```

## 배경

- 타입 코드는 프로그래밍 언어에 따라 열거형이나 심볼, 문자열, 숫자 등으로 표현하며, 외부 서비스가 제공하는 데이터를 다루려 할 때 달려오는 일이 흔하다.
- 서브 클래스는 두 가지 면에서 특히 매력적이다.
    - 조건에 따라 다르게 동작하도록 해주는 다형성을 제공한다.
    - 매력은 특정 타입에서만 의미가 있는 값을 사용하는 필드나 메서드가 있을 때 발현된다.
- 클래스에 직접 적용할지, 아니면 타입 코드 자체에 적용할지를 고민해야 한다.

## 절차

1. 타입 코드 필드를 자가 캡슐화한다.
2. 타입 코드 값 하나를 선택하여 그 값에 해당하는 서브 클래스를 만든다. 타입 코드 게터 메서드를 오버라이드하여 해당 타입 코드의 리터럴 값을 반환하게 한다.
3. 매개변수로 받은 타입 코드와 방금 만든 서브 클래스를 매핑하여 선택 로직을 만든다.
→ 직접 상속일 때는 생성자를 팩터리 함수로 바꾸기를 적용하고 선택 로직을 팩터리에 넣는다. 간접 상속일 때는 선택 로직을 생성자에 두면 될 것이다.
4. 테스트한다.
5. 타입 코드 값 각각에 대해 서브 클래스 생성과 선택 로직 추가를 반복한다. 클래스 하나가 완성될 때마다 테스트한다.
6. 타입 코드 필드를 제거한다.
7. 테스트한다.
8. 타입 코드 접근자를 이용하는 메서드 모두에 메서드 내리기와 조건부 로직을 다형성으로 바꾸기를 적용한다.

# 12.7 서브클래스 제거하기
- 다형성이나 유즈케이스가 없다면 서브클래스를 만들 필요가 없다. 
```jsx
class Person {
	get genderCode() {return "X";}
}
class Male extends Person {
	get genderCode() {return "M";}
}
class Female extends Person {
	get genderCode() {return "F";}
}
```

```jsx
class Person {
	get genderCode() {
		return this._genderCode;
	}
}
```

## 배경

- 서브클래싱은 원래 데이터 구조와는 다른 변종을 만들거나 종류에 따라 동작이 달라지게 할 수 있는 유용한 메커니즘이다.

## 절차

1. 서브 클래스의 생성자를 팩터리 함수로 바꾼다.
→ 생성자를 사용하는 측에서 데이터 필드를 이용해 어떤 서브 클래스를 생성할지 결정한다면 그 결정 로직을 슈퍼클래스의 팩터리 매서드에 넣는다.
2. 서브 클래스의 타입을 검사하는 코드가 있다면 그 검사 코드에 함수 추출하기와 함수 옮기기를 차례로 적용하여 슈퍼클래스로 옮긴다. 하나 변경할 때마다 테스트한다.
3. 서브 클래스의 타입을 나타내는 필드를 슈퍼클래스에 만든다.
4. 서브 클래스를 참조하는 메서드가 방금 만든 타입 필드를 이용하도록 수정한다.
5. 서브 클래스를 지운다.
6. 테스트한다.

# 12.8 슈퍼클래스 추출하기

```jsx
class Department {
	get totalAnnualCost() {...}
	get name() {...}
	get headCount() {...}
}

class Employee {
	get annualCost() {...}
	get name() {...}
	get id() {...}
}
```

```jsx
class Party {
	get name() {...}
	get annualCost() {...}
}

class Department extends Party {
	get annualCost() {...}
	get headCount() {...}
}

class Employee extends Party {
	get annualCost() {...}
	get id() {...}
}
```

## 배경

- 비슷한 일을 수행하는 두 클래스가 보이면 상속 메커니즘을 이용해서 비슷한 부분을 공통의 슈퍼클래스로 옮겨 담을 수 있다.
- 상속은 프로그램이 성장하면서 깨우쳐가게 되며, 슈퍼클래스로 끌어올리고 싶은 공통 요소를 찾았을 때 수행하는 사례가 잦았다.
- 대안으로는 클래스 추출하기가 있다.

## 절차

1. 빈 슈퍼클래스를 만든다. 원래의 클래스들이 새 클래스를 상속하도록 한다.
→ 필요하다면 생성자에 함수 선언 바꾸기를 적용한다.
2. 테스트한다.
3. 생성자 본문 올리기, 메서드 올리기, 필드 올리기를 차례로 적용하여 공통 원소를 슈퍼클래스로 옮긴다.
4. 서브 클래스에 남은 메서드들을 검토한다. 공통되는 부분이 있다면 함수로 추출한 다음 메스드 올리기를 적용한다.
5. 원래 클래스들을 사용하는 코드를 검토하여 슈퍼클래스의 인터페이스를 사용하게 할지 고민해본다. 

# 12.9 계층 합치기

```jsx
class Employee {...}
class Salesperson extends Employee {...}
```

```jsx
class Employee {...}
```

## 배경

- 클래스 계층구조를 리팩터링하다 보면 기능들을 위로 올리거나 아래로 내리는 일은 다반사로 벌어진다.
- 어떤 클래스와 그 부모가 너무 비슷해져서 더는 독립적으로 존재해야 할 이유가 사라진다.

## 절차

1. 두 클래스 중 제거할 것을 고른다.
→ 미래를 생각하여 더 적합한 이름의 클래스를 남기자. 둘 다 적절치 않다면 임의로 하나로 고른다.
2. 필드 올리기와 메서드 올리기 혹은 필드 내리기와 메서드 내리기를 적용하여 모든 요소를 하나의 클래스로 옮긴다. 
3. 제거할 클래스를 참조하던 모든 코드가 남겨질 클래스를 참조하도록 고친다.
4. 빈 클래스를 제거한다.
5. 테스트한다.

# 12.10 서브클래스를 위임으로 바꾸기

```jsx
class Order {
	get daysToShip() {
		return this._warehouse.daysToShip;
	}
}

class PriorityOrder extents Order {
	get daysToShip() {
		return this._priorityPlan.daysToShip;
	}
}
```

```jsx
class Order {
	get daysToShip() {
		return (this._priorityDelegate)
			? tihs._priorityDelegate.daysToShip
			: this._warehouse.dysToShip;
	}
}

class PriorityOrder extents Order {
	get daysToShip() {
		return this._priorityPlan.daysToShip;
	}
}
```
## 다른 예제
- 상속
   - 다중 상속이 허용이 안되는 언어가 있다. 
	 - 수정, 확장이 어렵다. 
	 - 수직적인 관계를 이용해서 코드를 재사용, 확장
	 - 자동적으로 부모의 함수가 추가 되기 때문에 의미가 않맞는 경우가 있을 수 있다.
	 - 수정이 어렵고 유지보수가 어려워지는 문제점이 있다.
- 위임
   - 레고를 조립하듯 사용할 수 있다.
	 - 필요한 부품을 주입받아서 활용.
```js
// 상속
class Printer {
	print() {
		console.log('기본적인 출력');
	}
}

class RedPrinter extends Printer{
	print(){
		console.log('🔴 출력!')
	}
}

const printers [new Printer(), new RedPrinter()];
printers.forEach((printer) => printer.print());
```
```js
// 위임
class Printer {
	#printerDelegate;
	constructor(printerDelegate){
		this.#printerDelegate = printerDelegate;
	}
	print() {
		this.#printerDelegate 
			? this.#printerDelegate.print() 
			: console.log('기본적인 출력');
	}
}

class RedPrinter {
	print(){
		console.log('🔴 출력!')
	}
}

const printers [new Printer(), new Printer(new RedPrinter())];
printers.forEach((printer) => printer.print());
```
```ts
// 타입스크립트 위임
interface PrinterDelegate{
	print(): void
}

class Printer {
	private printerDelegate: PrinterDelegate;
	constructor(printerDelegate?: PrinterDelegate){
		this.printerDelegate = printerDelegate
			? printerDelegate
			: new DefaultPrinter();
	}
	print() {
		this.printerDelegate 
			? this.printerDelegate.print() 
			: console.log('기본적인 출력');
	}
}
class DefaultPrinter implements PrinterDelegate{
	print(){
		console.log('출력!')
	}
}

class RedPrinter implements PrinterDelegate{
	print(){
		console.log('🔴 출력!')
	}
}

class BlackPrinter implements PrinterDelegate{
	print(){
		console.log('⚫️ 출력!')
	}
}

const printers [new Printer(), new Printer(new RedPrinter()), new Printer(new BlackPrinter())];
printers.forEach((printer) => printer.print());
```

## 배경

- 속한 갈래에 따라 동작이 달라지는 객체들은 상속으로 표현하는 게 자연스럽다.
- 공통 데이터와 동작은 모두 슈퍼클래스에 두고 서브 클래스는 자신에 맞게 기능을 추가하거나 오버라이드하면 된다.

## 절차

1. 생성자를 호출하는 곳이 많다면 생성자를 팩터리 함수로 바꾼다.
2. 위임으로 활용할 빈 클래스를 만든다. 이 클래스의 생성자는 서브클래스에 특화된 데이터를 전부 받아야 하며, 보통은 슈퍼클래스를 가리키는 역참조도 필요하다.
3. 위임을 저장할 필드를 슈퍼클래스에 추가한다.
4. 서브 클래스 생성 코드를 수정하여 위임 인스턴스를 생성하고 위임 필드에 대입해 초기화한다.
→ 이 작업은 팩터리 함수가 수행한다. 혹은 생성자가 정확한 위임 인스턴스를 생성할 수 있는 게 확실하다면 생성자에서 수행할 수도 있다.
5. 서브 클래스의 매서드 중 위임 클래스로 이동할 고른다.
6. 함수 옮기기를 적용해 위임 클래스로 옮긴다. 원래 메서드에서 위임하는 코드는 지우지 않는다.
→ 이 메서드가 사용하는 원소 중 위임으로 옮겨야 하는 게 있다면 함께 옮긴다. 슈퍼클래스에 유지해야 할 원소를 참조한다면 슈퍼클래스를 참조하는 필드를 위힘에 추가한다.
7. 서브 클래스 외부에도 원래 메서드를 호출하는 코드가 있다면 서브 클래스의 위임 코드를 슈퍼 클래스로 옮긴다. 이때 위임이 존재하는지를 검사하는 보호 코드로 감싸야 한다. 호출하는 외부 코드가 없다면 원래 메서드는 죽은 코드가 되므로 제거한다.
→ 서브 클래스가 둘 이상이고 서브클래스들에서 중복이 생겨나기 시작했다면 슈퍼클래스를 추출한다. 이렇게 하여 기본 동작이 위임 슈퍼클래스로 옮겨졌다면 슈퍼클래스의 위임 메서드들에는 보코 코드가 필요 없다.
8. 테스트한다.
9. 서브 클래스의 모든 메서드가 옮겨질 때까지 5~8 과정을 반복한다.
10. 서브 클래스들의 생성자를 호출하는 코드를 찾아서 슈퍼클래스의 생성자를 사용하도록 수정한다.
11. 테스트한다.
12. 서브 클래스를 삭제한다.(죽은 코드 제거하기)

# 12.11 슈퍼클래스를 위임으로 바꾸기

```jsx
class List {...}
class Stack extends List {...}
```

```jsx
class Stack {
	constructor() {
		this._storage = new List();
	}
}

class List {...}
```

## 배경

- 슈퍼클래스의 기능들이 서브 클래스에는 어울리지 않는다면 그 기능들을 상속을 통해 이용하면 안된다는 신호다.
- 슈퍼 클래스가 사용 되는 모든 곳에서 서브클래스의 인스턴스를 대신 사용해도 이상없이 동작해야한다.
- 서브 클래스 방식 모델링이 합리적일 때라도 슈퍼클래스를 위임으로 바꾸기도 한다.
- 위임의 기능을 이용할 호스트의 함수 모두 전달함수로 만들어야 하는 단점도 있다.
- 웬만하면 상속을 먼저 사용하고 나중에 문제가 생기면 슈퍼클래스를 위임으로 바꾸라는 것이 조언이다.

## 절차

1. 슈퍼클래스 객체를 참조하는 필드를 서브 클래스에 만든다(이번 리팩터링을 끝마치면 슈퍼클래스가 위임 객체가 될 것이므로 이 필드를 `위임 참조`라 부르자). 위임 참조를 새로운 슈퍼클래스 인스턴스로 초기화한다.
2. 슈퍼 클래스의 동작 각각에 대응하는 전달 함수를 서브 클래스에 만든다(물론 위임 참조로 전달한다). 서로 관련된 함수끼리 그룹으로 묶어 진행하며, 그룹을 하나씩 만들 때마다 테스트한다.
→ 대부분은 전달 함수 각각을 테스트할 수 있을 것이다. 하지만 예컨대 게터와 세터 쌍은 둘 다 옮긴 후에야 테스트할 수 있다. 
3. 슈퍼클래스의 동작 모두가 전달 함수로 오버라이드 되었다면 상속 관계를 끊는다.