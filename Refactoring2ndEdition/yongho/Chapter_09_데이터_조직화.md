# 9.1 변수 쪼개기

```jsx
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```

```jsx
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

## 배경

- 긴 코드의 결과를 저장했다가 나중에 쉽게 참조하려는 목적으로 사용하는 변수는 값을 단 한 번만대입해야 한다.
- 대입이 두 번 이상 이뤄진다면 여러 가지 역할을 수행한다는 신호다.
- 역할이 둘 이상인 변수가 있다면 쪼개야 한다.

## 절차

1. 변수를 선언한 곳과 값을 처음으로 대입하는 곳에서 변수 이름을 바꾼다.
→ 이후의 대입이 항상 `i=i+<무언가>` 형태라면 수집 변수이므로 쪼개면 안 된다. 수집 변수는 총합 계산, 문자열 연결, 스트림에 쓰기, 컬렉션에 추가하기 등의 용도로 흔히 쓰인다.
2. 가능하면 이대 불편으로 선언한다.
3. 이 변수에 두 번째로 값을 대입하는 곳 앞까지의 모든 참조(이 변수가 쓰인 곳)를 새로운 변수 이름으로 바꾼다.
4. 두 번째 대입 시 변수를 원래 이름으로 다시 선언한다. 
5. 테스트한다. 
6. 반복한다. 매 반복에서 변수를 새로운 이름으로 선언하고 다음번 대입 때까지의 모든 참조를 새 변수명으로 바꾼다. 이 과정을 마지막 대입까지 반복한다.

# 9.2 필드 이름 바꾸기

```jsx
class Organization {
	get name () { ... }
}
```

```jsx
class Organization {
	get title () { ... }
}
```

## 배경

- 이름은 중요하다.
- 데이터 구조는 무슨 일이 벌어지는지를 이해하는 열쇠다.

## 절차

1. 레코드의 유효 범위가 제한적이라면 필드에 접근하는 모든 코드를 수정한 후 테스트한다. 이후 단계는 필요 없다.
2. 레코드가 캡슐화되지 않았다면 우선 레코드를 캡슐화한다.
3. 캡슐화된 객체 안의 `private` 필드명을 변경하고, 그에 맞게 내부 메서드들을 수정한다.
4. 테스트한다.
5. 생성자의 매개변수 중 필드와 이름이 겹치는 게 있다면 함수 선언 바꾸기로 변경한다.
6. 접근자들의 이름도 바꿔준다.

# 9.3 파생 변수를 질의 함수로 바꾸기

```jsx
get discountedTotal() { return this._discaountedTotal; }
set discount(aNumber){
	const old = this._discaount;
	this._discount = aNumber;
	this._discountedTotal += old - aNumber;
}
```

```jsx
get discountedTotal() { return this._baseTotal - this._discount }
set discount(aNumber) { this._discount = aNumber }
```
```js
// 수정 전
class ProductionPlan {
  // 다른 코드 있다고 가정
  get production() {
    return this._production;
  }
  applyAdjustment(adjustment) {
    this._adjustments.push(adjustment);
    this._production += adjustment.amount;
  }
}

// 수정 수
class ProductionPlan {
  // 다른 코드 있다고 가정
  get production() {
    return this._adjustments.reduce((sum, a) => sum + a.amount, 0);
  }
  applyAdjustment(adjustment) {
    this._adjustments.push(adjustment);
  }
}

```

## 배경

- 가변 데이터는 소프트웨어 문제를 일으키는 가장 큰 골칫거리에 속한다.
- 가변 데이터의 유효 범위를 가능한 한 좁혀야 한다.
- 값을 쉽게 계산해낼 수 있는 변수들을 모두 제거
- 새로운 데이터 구조를 생성하는 변형 연산이라면 비록 계산 코드로 대체할 수 있더라도 그대로 두는 것도 좋다.
- 변형 연산의 두 가지
    1. 데이터 구조를 감싸며 그 데이터에 기초하여 계산한 결과를 속성으로 제공하는 객체
    2. 데이터 구조를 받아 다른 데이터 구조로 변환해야 하는 함수

## 절차

1. 변수 값이 갱신되는 지점을 모두 찾는다. 필요하면 변수 쪼개기를 활용해 각 갱신 지점에서 변수를 분리한다. 
2. 해당 변수의 값을 계산해주는 함수를 만든다.
3. 해당 변수가 사용되는 모든 곳에 어서션을 추가하여 함수의 계산 결과가 변수의 값과 같은지 확인한다.
→ 필요하면 변수 캡슐화하기를 적용하여 어서션이 들어갈 장소를 마련해준다.
4. 테스트한다.
5. 변수를 읽는 코드를 모두 함수 호출로 대체한다.
6. 테스트한다.
7. 변수를 선언하고 갱신하는 코드를 죽은 코드 제거하기로 없앤다. 

# 9.4 참조를 값으로 바꾸기

```jsx
class Product {
	applyDiscount(arg) { this._price.amount -= arg; }
```

```jsx
class Product {
	applyDiscount(arg) {
		this._price = new Money(this._price.amount - arg, this._price.currency);
	}
```

```js
// 수정 전
class Person {
  #name;
  #telephoneNumber;
  constructor(name, areaCode, number) {
    this.#name = name;
    this.#telephoneNumber = new TelephoneNumber(areaCode, number);
  }

  get name() {
    return this.#name;
  }

  set name(arg) {
    this.#name = arg;
  }

  get telephoneNumber() {
    return this.#telephoneNumber.toString;
  }

  get officeAreaCode() {
    return this.#telephoneNumber.areaCode;
  }

  set officeAreaCode(value) {
    this.#telephoneNumber.areaCode = value;
  }

  get officeNumber() {
    return this.#telephoneNumber.number;
  }

  set officeNumber(value) {
    this.#telephoneNumber.number = value;
  }
}

class TelephoneNumber {
  #areaCode;
  #number;
  constructor(area, number) {
    this.#areaCode = area;
    this.#number = number;
  }

  get areaCode() {
    return this.#areaCode;
  }
  set areaCode(arg) {
    this.#areaCode = arg;
  }

  get number() {
    return this.#number;
  }
  set number(arg) {
    this.#number = arg;
  }

  get toString() {
    return `(${this.#areaCode}) ${this.#number}`;
  }
}

// 수정 후 
class Person {
  #name;
  #telephoneNumber;
  constructor(name, areaCode, number) {
    this.#name = name;
    this.#telephoneNumber = new TelephoneNumber(areaCode, number);
  }

  get name() {
    return this.#name;
  }

  set name(arg) {
    this.#name = arg;
  }

  get telephoneNumber() {
    return this.#telephoneNumber.toString;
  }

  get officeAreaCode() {
    return this.#telephoneNumber.areaCode;
  }

  set officeAreaCode(value) {
    this.#telephoneNumber = new TelephoneNumber2(value, this.officeNumber);
  }

  get officeNumber() {
    return this.#telephoneNumber.number;
  }

  set officeNumber(value) {
    this.#telephoneNumber = new TelephoneNumber2(this.areaCode, value);
  }
}

class TelephoneNumber {
  #areaCode;
  #number;
  constructor(area, number) {
    this.#areaCode = area;
    this.#number = number;
  }

  get areaCode() {
    return this.#areaCode;
  }

  get number() {
    return this.#number;
  }

  get toString() {
    return `(${this.#areaCode}) ${this.#number}`;
  }
}

```

## 배경

- 객체(데이터 구조)를 다른 객체(데이터 구조)에 중첩하면 내부 객체를 참조 혹은 값으로 취급할 수 있다.
- 참조냐 값이냐의 차이는 내부 객체의 속성을 갱신하는 방식에서 가장 극명하게 드러난다.
    - 참조: 내부 객체는 그대로 둔 채 그 객체의 속성만 갱신
    - 값: 새로운 속성을 담은 객체로 기존 내부 객체를 통째로 대체
- 필드를 값으로 다룬다면 내부 객체의 클래스를 수정하여 값 객체로 만들 수 있다.

## 절차

1. 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인한다.
2. 각각의 세터를 하나씩 제거한다.
3. 이 값 객체의 필드들을 사용하는 동치성 비교 메서드를 만든다.
→ 대부분의 언어는 이런 상황에 사용할 수 있도록 오버라이딩 가능한 동치성 비교 메서드를 제공한다. 동치성 비교 메서드를 오버라이드할 때는 보통 해시코드 생성 메서드도 함께 오버라이드해야한다.

# 9.5 값을 참조로 바꾸기

```jsx
let customer = new Customer (customerData);
```

```jsx
let customer = customerRepository.get(customerData.id)
```

```js
// 수정 전
class Order {
  constructor(data) {
    this._number = data.number;
    this._customer = new Customer(data.customerId);
  }

  get customer() {
    return this._customer;
  }
}

class Customer {
  constructor(id) {
    this._id = id;
  }

  get id() {
    return this._id;
  }
}
/**
 * 불변성은 좋다.
 * 하지만 계속된 정보가 업데이트 될 때
 * 이전에 생성한 인스턴스의 정보가 업데이트가 되어야 한다
 * 그래서 값을 참조로 바꾼다.
 */
// 수정 후
const customerRepository = new CustomerRepository()

const order = new Order2(data.number, customerRepository.registorCustomer(data.customerId))

class Order2 {
  constructor(number, customer) {
    this._number = data.number;
    this._customer = customer;
  }

  get customer() {
    return this._customer;
  }
}

class Customer2 {
  constructor(id, name) {
    this._id = id;
    this._name = name;
  }

  get id() {
    return this._id;
  }
}

class CustomerRepository {
  #customers;

  constructor() {
    this.#customers = new Map();
  }

  registorCustomer(id) {
    if(!this.#customers.has(id)){
      this.#customers.set(id, new Customer(id));
    }
    return findCustomer(id);
  }

  findCustomer(id) {
    return this.#customers.get(id);
  }
}

//리파지토리 패턴을 사용

```

## 배경

- 하나의 데이터 구조 안에 논리적으로 똑같은 제3의 데이터 구조를 참조하는 레코드가 여러 개 있을 때가 있다.
- 논리적으로 같은 데이터를 물리적으로 복제해 사용할 때 가장 크게 문제 되는 상황은 그 데이터를 갱신해야 할 때다.

## 절차

1. 같은 부류에 속하는 객체들을 보관할 저장소를 만든다. (이미 있다면 생략)
2. 생성자에서 이 부류의 객체들 중 특정 객체를 정확히 찾아내는 방법이 있는지 확인한다.
3. 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾도록 한다. 하나 수정할 때마다 테스트한다. 

# 9.6 매직 리터럴 바꾸기

```jsx
function potentialEnergy (mass, height) {
	return mass * 9.81 * height;
}
```

```jsx
const STANDARD_GRAVITY = 9.81;
function potentialEnergy (mass, height) {
	return mass * STANDARD_GRAVITY * height;
}
```

## 배경

- 매직 리터럴이란 소스 코드에 (보통 여러 곳에) 등장하는 일반적인 리터럴 값을 말한다.
- 코드 자체가 뜯을 분명하게 드러내는 게 좋다.
- 상수를 정의하고 숫자 대신 상수를 사용하도록 바꾸면 될 것이다.

## 절차

1. 상수를 선언하고 매직 리터럴을 대입한다.
2. 해당 리터럴이 사용되는 곳을 모두 찾는다.
3. 찾은 곳 각각에서 리터럴이 새 상수와 똑같은 의미로 쓰였는지 확인하여, 같은 의미라면 상수로 대체한 후 테스트한다.