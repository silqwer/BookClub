모듈과 함수는 소프트웨어를 구성하는 빌딩 블록이며, API는 이 블록들을 끼워 맞추는 연결부다. 

좋은 API는 데이터를 갱신하는 함수와 그저 조회만 하는 함수를 명확히 구분한다. 

# 11.1 질의 함수와 변경함수 분리하기

```jsx
function getTotalOutstandingAndSendBill() {
	const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
	sendBill();
	return result;
}
```

```jsx
function totalOutstanding() {
	return customer.invoices.reduce((total, each) => each.amount + total, 0);
}
function sendBill() {
	emailGateway.send(formatBill(customer));
}
```
## 다른 예제
- 두 가지 동작을 수행 한다. 
- 알람을 찾는 동작과 알람을 보내는 동작
- 찾는다면 질의용 함수를 만들어 재사용성을 높이자.
```js
export function alertForMiscreant(people, alarm) {
  for (const p of people) {
    if (p === 'Don') {
      setOffAlarms(alarm, p);
      return 'Don';
    }
    if (p === 'John') {
      setOffAlarms(alarm, p);
      return 'John';
    }
  }
  return '';
}

function setOffAlarms(alarm, p) {
  alarm.setOff('Found Miscreant ' + p);
}
```
```js
export function alertForMiscreant(people, alarm) {
  const miscreant = findMiscreant(people);
  setOffAlarms(alarm, miscreant)
}

function findMiscreant(people){
  for (const p of people) {
    if (p === 'Don') {
      setOffAlarms(alarm, p);
      return 'Don';
    }
    if (p === 'John') {
      setOffAlarms(alarm, p);
      return 'John';
    }
  }
  return '';
}

function setOffAlarms(alarm, p) {
  alarm.setOff('Found Miscreant ' + p);
}
```

## 배경

- 외부에서 관찰할 수 있는 겉보기 부수 효과가 전혀 없이 값을 반환해주는 함수를 추구해야 한다.
- 겉보기 부수 효과가 있는 함수와 없는 함수는 명확히 구분하는 것이 좋다.
- 질의함수(읽기 함수)는 모두 부수 효과가 없어야 한다.
- 부수 효과가 있는 함수를 발견하면 상태를 변경하는 부분과 질의하는 부분을 분리한다.

## 절차

1. 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다.
→ 함수 내부를 살펴 무엇을 반환하는지 찾는다. 어떤 변수의 값을 반환한다면 그 변수 이름이 훌륭한 단초가 될 것이다.
2. 새 질의 함수에서 부수 효과를 모두 제거한다.
3. 정적 검사를 수행한다.
4. 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아낸다. 호출하는 곳에서 반환 값을 사용한다면 질의 함수를 호출하도록 바꾸고, 원래 함수를 호출하는 코드를 바로 아래 줄에 새로 추가한다. 하나 수정할 때마다 테스트한다. 
5. 원래 함수에서 질의 관련 코드를 제거한다.
6. 테스트한다

# 11.2 함수 배개변수화하기

```jsx
function tenPercentRaise(aPerson) {
	aPerson.salary = aPerson.salary.multiply(1.1);
} 
function fivePercentRaise(aPerson) {
	aPerson.salary = aPerson.salary.multiply(1.05);
}
```

```jsx
function raise(aPerson, factor) {
	aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```
## 다른 예제 
```js
export function baseCharge(usage) {
  if (usage < 0) return usd(0);
  const amount =
    bottomBand(usage) * 0.03 + middleBand(usage) * 0.05 + topBand(usage) * 0.07;
  return usd(amount);
}

function bottomBand(usage) {
  return Math.min(usage, 100);
}

function middleBand(usage) {
  return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}

function topBand(usage) {
  return usage > 200 ? usage - 200 : 0;
}

function usd(value) {
  return {
    currency: '$',
    currencyName: 'USD',
    value: value,
  };
}
```
```js
export function baseCharge(usage) {
  if (usage < 0) return usd(0);
  const amount =
  withinBand(usage, 0, 100) * 0.03 + withinBand(usage, 100, 200) * 0.05 + withinBand(usage, 200, Infinity) * 0.07;
  return usd(amount);
}

function withinBand(usage, bottom, top){
  return usage > bottom ? Math.min(usage, top) - bottom : 0;
}

function usd(value) {
  return {
    currency: '$',
    currencyName: 'USD',
    value: value,
  };
}

/**
 * Math.min(200, Infinity)
 * 200
 * Infinity: 양의 무한대 
 */

```

## 배경

- 두 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면, 그 다른 값만 매개변수로 받아 처리하는 함수 하나로 합쳐서 중복을 없앨 수 있다.

## 절차

1. 비슷한 함수 중 하나를 선택한다. 
2. 함수 선언 바꾸기로 리터럴들을 매개변서로 추가한다.
3. 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다.
4. 테스트한다.
5. 매개 변수로 받은 값을 사용하도록 함수 본문을 수정한다. 하나 수정할 때마다 테스트한다. 
6. 비슷한 다른 함수를 호출하는 코드를 찾아 매개 변수화된 함수를 호출하도록 하나씩 수정한다. 하나 수정할 때마다 테스트한다. 
→ 매개 변수화된 함수가 대체할 비슷한 함수와 다르게 동작한다면, 그 비슷한 함수의 동작도 처리할 수 있도록 본문 코드를 적절히 수정한 후 진행한다. 

# 11.3 플래그 인수 제거하기

```jsx
function setDimension(name, value) {
	if (name === "height") {
		this._height = value;
		return;
	}
	if (name === "width") {
		this._width = value;
		return
	}
}
```

```jsx
function setHeight(value) {this._height = value;}
function setWidth(value) {this._width = value;}
```
## 다른 예제 1
- 다른 인자를 기준으로 다른 행동을 수행하게 하는 함수는 지양하자.
- 
```js
class Concert {
  book(customer, isPremium) {}
}
```
```js
class Concert {
	regularBook(customer) {}
	premiumBook(customer) {}

	// 내부적으로 중복되는 코드가 너무 많고 정말정말 필요하다면
	#book(customer, isPremium) {}
}
```
## 다른 예제 2
- 가장 좋은 함수는 매개변수가 없는 함수
```js
function setSwitch(on);
```
```js
function setSwitchOn();
function setSwitchOff();
```
## 배경

- 플래그 인수란 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수다.
- 호출할 수 있는 함수들이 무엇이고 어떻게 호출해야 하는지를 이해하기가 어려워진다.
- 플래그 인수를 제거하면 코드가 깔끔해진다.
- 코드 분석 도구는 프리미엄 로직 호출과 일반 로직 호출 차이를 더 쉽게 파악할 수 있게 된다.

## 절차

1. 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적  함수들을 생성한다. 
→ 주가 되는 함수에 깔끔한 분배 조건문이 포함되어 있다면 조건문 분해하기로 명시적함수들을 생성하자. 그렇지 않다면 래핑 함수 형태로 만든다. 
2. 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정한다.

# 11.4 객체 통째로 넘기기

```jsx
const low = aRoom.daysTempRange.low;
const high = aRoome.daysTempRange.high;
if(aPlan.withinRange(low, high))
```

```jsx
if (aPlan.withinRange(aRoom.daysTempRange))
```
## 다른 예제
- 객체를 넘길 때는 간편할 수 있으나 의존성을 가져도 되는지 커플링(결합도)이 되지 않는지 고민해야한다.
```js
export function temperatureAlerts(room, plan) {
  const alerts = [];
  const low = room.daysTempRange.low;
  const high = room.daysTempRange.high;
  if (!plan.withinRange(low, high)) {
    alerts.push('room temperature went outside range');
  }

  return alerts;
}

export class HeatingPlan {
  constructor(temperatureRange) {
    this._temperatureRange = temperatureRange;
  }

  withinRange(bottom, top) {
    return (
      bottom >= this._temperatureRange.low && top <= this._temperatureRange.high
    );
  }
}
```
```js
export function temperatureAlerts(room, plan) {
  const alerts = [];

  if (!plan.withinRange(room.daysTempRange)) {
    alerts.push('room temperature went outside range');
  }

  return alerts;
}

export class HeatingPlan {
  constructor(temperatureRange) {
    this._temperatureRange = temperatureRange;
  }

  withinRange(range) {
    return (
      range.bottom >= this._temperatureRange.low && range.top <= this._temperatureRange.high
    );
  }
}
```
## 배경

- 레코드를 통째로 넘기면 변화에 대응하기 쉽다.

## 절차

1. 매개변수들을 원하는 형태로 받는 빈 함수를 만든다.
→ 마지막 단계에서 이 함수의 이름을 변경해야 하니 검색하기 쉬운 이름으로 지어준다.
2. 새 함수의 본문에서는 원래 함수를 호출하도록 하여, 새 매개변수와 원래 함수의 매개변수를 매핑한다.
3. 정적 검사를 수행한다.
4. 모든 호출자가 새 함수를 사용하게 수정한다. 하나씩 수정하며 테스트하자.
→ 수정 후에는 원래의 매개변수를 만들어내는 코드 일부가 필요 없어질 수 있다. 따라서 죽은 코드 제거하기로 없앨 수 있을 것이다.
5. 호출자를 모두 수정했다면 원래 함수를 인라인 한다.
6. 새 함수의 이름을 적절히 수정하고 모든 호출자에 반영한다.

# 11.5 매개변수를 질의 함수로 바꾸기

```jsx
availableVacation(anEmployee, anEmployee.grade);
function availableVacation(anEmployee, grade){ 
	// 연휴 계산
```

```jsx
availableVacation(anEmployee)

function availableVacation(anEmployee) {
	const grade = anEmployee.grade;
	// 연휴 계산 ...
```
## 다른 예제
- 클래스, 모듈 안에서 사용하는 함수를 게터로 올리면 가독성, 유지보수성이 높아진다.
- 매개변수를 피할 수 있으면 피하자.
- 질의함수로 충분히 값을 가져올 수 있다면 클래스의 get을 이용해서 가져오는 것이 깔끔하다.
```js
export class Order {
  constructor(quantity, itemPrice) {
    this.quantity = quantity;
    this.itemPrice = itemPrice;
  }

  get finalPrice() {
    const basePrice = this.quantity * this.itemPrice;
    let discountLevel;
    if (this.quantity > 100) discountLevel = 2;
    else discountLevel = 1;
    return this.discountedPrice(basePrice, discountLevel);
  }

  discountedPrice(basePrice, discountLevel) {
    switch (discountLevel) {
      case 1:
        return basePrice * 0.95;
      case 2:
        return basePrice * 0.9;
    }
  }
}
```
```js
export class Order {
  constructor(quantity, itemPrice) {
    this.quantity = quantity;
    this.itemPrice = itemPrice;
  }
	get basePrice() {
		return this.quantity * this.itemPrice;
	}
  get finalPrice() {
    return this.discountedPrice();
  }

	get discountLevel() {
		return this.quantity > 100 ? 2 : 1;
	}

  discountedPrice() {
    switch (this.discountLevel) {
      case 1:
        return this.basePrice * 0.95;
      case 2:
        return this.basePrice * 0.9;
    }
  }
}
```
## 배경

- 매개변수 목록은 함수의 변동 요인을 모아놓은 곳이다.
- 함수의 동작에 변화를 줄 수 있는 일차적인 수단이다.
- 매개변수를 질의 함수로 바꾸지 말아야 할 상황은 매개변수를 제거하면 피 호출 함수에 원치 않는 의존성이 생길 데다.

## 절차

1. 필요하시다면 대상 매개변수의 값을 계산하는 코드를 별도 함수로 추출해놓는다.
2. 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다. 하나 수정할 때마다 테스트한다.
3. 함수 선언 바꾸기로 대상 매개변수를 없앤다. 

# 11.6 질의 함수를 매개변수로 바꾸기
- 다른 모듈, 외부 전역변수, 클래스, 객체, 함수에 있는 것을 내부에서 접근하는 것은 좋지 않다.
- 응집도가 높은 상태에서는 외부 데이터를 접근 하는것이 아니라 전달 받을 수 있도록 하는 것이 좋다.
```jsx
targetTemperature(aPlan)

function targetTemperature(aPlan) {
	currentTemperature = thermostat.currentTemperature;
	// 생략
```

```jsx
targetTemperature(aPlan, thermostat.currentTemperature);

function targetTemperature(aPlan, currentTemperatrue) {
	// 생략
```

## 배경

- 코드의 의존 관계를 바꾸려 할 때 벌어진다.

## 절차

1. 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리한다.
2. 함수 본문 주 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출한다.
→ 이 함수의 이름은 나중에 수정해야 하니 검색하니 쉬운 이름으로 짓는다. 
3. 방금 만든 변수를 인라인 하여 제거한다.
4. 원래 함수도 인라인 한다.
5. 새 함수의 이름을 원래 함수의 이름으로 고쳐준다.

# 11.7 새터 제거하기
- 데이터가 읽기가 가능한지 쓰기가 가능한지 파악하고 캡슐화를 하자.

```jsx
class Person{
	get name() {...}
	set name(aString) {...}
```

```jsx
class Person {
	get name() {...}
```

## 배경

- 객체 생성 후에는 조정되지 않길 원하는 필드라면 세터를 제공하지 않았을 것이다.
- 세터 제거하기 리팩터링이 필요한 상황은 두 가지다.
    - 사람들이 무조건 점근자 메서드를 통해서만 필드를 다루려 할 때다.
    - 클라이언트에서 생성 스크립트를 사용해 객체를 생성할 때다.

## 절차

1. 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다(함수 선언 바꾸기). 그런 다음 생성자 안에서 적절한 세터를 호출한다.
→ 세터 여러 개를 제거하라면 해당 값 몯를 한꺼번에 생성자에 추가한다. 그러면 이후 과정이 간소해진다.
2. 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 때마다 테스트한다.
→ (갱신하려는 대상이 공유 참조 객체라서) 새로운 객체를 생성하는 방식으로는 세터 호출을 대처할 수 없다면 이 리팩터링을 취소한다.
3. 세터 메서드를 인라인 한다. 가능하다면 해당 필드를 불변으로 만든다.
4. 테스트한다. 

# 11.8 생성자를 팩터리 함수로 바꾸기

```jsx
leadEngineer = new Employee(document.leadEngineer, 'E');
```

```jsx
leadEngineer = createEngineer(document.leadEngineer);
```
## 다른 예제
- 어떤 클래스를 만들때 생성자를 호출하는 방식이 복잡하다면 인스턴스를 만드는 로직도 캡슐화 할 수 있다. 
- 자바스크립트에서는 생성자를 `private, #`으로 만들 수가 없다. 
- `static`함수로 인스턴스를 생성하는 팩터리 함수를 만든다.
- 생성자를 `private`로 만들 수 없으니 별도로 Factory Class 를 만들어서 사용해야 할까?
```js
export class Employee {
  constructor(name, typeCode) {
    this._name = name;
    this._typeCode = typeCode;
  }
  get name() {
    return this._name;
  }

  get type() {
    return Employee.legalTypeCodes[this._typeCode];
  }

  static get legalTypeCodes() {
    return { E: 'Engineer', M: 'Manager', S: 'Salesman' };
  }
}
```
```js
export class Employee {
  constructor(name, typeCode) {
    this._name = name;
    this._typeCode = typeCode;
  }
  get name() {
    return this._name;
  }

  get type() {
    return Employee.legalTypeCodes[this._typeCode];
  }

  static get legalTypeCodes() {
    return { E: 'Engineer', M: 'Manager', S: 'Salesman' };
  }

	static createEngineer(name){
		return new Employee(name, 'E')
	}

	static createManager(name){
		return new Employee(name, 'M')
	}

	static createSalesmanr(name){
		return new Employee(name, 'S')
	}
}

const employee = Employee.createEngineer('용호')

```
## 배경

- 팩터리 함수를 구현하는 과정에서 생성자를 호출할 수 있지만, 원한다면 다른 무언가로 대체할 수 있다.

## 절차

1. 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래의 생성자를 호출한다.
2. 생성자를 호출하던 코드를 팩터리 함수 호출로 바꾼다.
3. 하나씩 수정할 때마다 테스트한다.
4. 생성자의 가시 범위가 최소가 되도록 제한다.

# 11.9 함수를 명령으로 바꾸기

```jsx
function score(candidate, medicalExam, scoringGuide) {
	let result = 0;
	let healthLevel = 0;
	// 긴 코드 생략
}
```

```jsx
class Scorer {
	constructor(candidate, medicalExam, soringGuide) {
		this._candidate = candidate;
		this._medicalExam = medicalExam;
		this._scorinGuide = scoringGuide;
	}
	
	execute() {
		this._result = 0;
		this._healthLevel = 0;
		// 긴 코드 생략
	}
}
```
## 다른 예제
- 클래스 중에서 딱 한가지의 명령을 수행하는 함수, 클래스를 명령객체라고 한다. aka 커맨드 객체
   - 명령하는 동사 하나! 실행!
- 이렇게 만들어진 패턴을 커멘트 패턴이라고 한다. 
```js
export function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  let highMedicalRiskFlag = false;

  if (medicalExam.isSmoker) {
    healthLevel += 10;
    highMedicalRiskFlag = true;
  }
  let certificationGrade = 'regular';
  if (scoringGuide.stateWithLowCertification(candidate.originState)) {
    certificationGrade = 'low';
    result -= 5;
  }
  // lots more code like this
  result -= Math.max(healthLevel - 5, 0);
  return result;
}

export class ScoringGuide {
  stateWithLowCertification(state) {
    return state < 5;
  }
}
```
```js
export function score(candidate, medicalExam, scoringGuide) {
  return new Scorer(candidate, medicalExam, soringGuide).execute();
}
class Scorer {
	constructor(candidate, medicalExam, scoringGuide){
		this.candidate = candidate;
		this.medicalExam = medicalExam;
		this.scoringGuide = scoringGuide;
	}
	
	execute(){
		let result = 0;
		let healthLevel = 0;
		let highMedicalRiskFlag = false;

		if (this.medicalExam.isSmoker) {
			healthLevel += 10;
			highMedicalRiskFlag = true;
		}
		let certificationGrade = 'regular';
		if (this.scoringGuide.stateWithLowCertification(this.candidate.originState)) {
			certificationGrade = 'low';
			result -= 5;
		}
		// lots more code like this
		result -= Math.max(healthLevel - 5, 0);
		return result;
	}
}

export class ScoringGuide {
  stateWithLowCertification(state) {
    return state < 5;
  }
}
```
## 배경

- 함수를 그 함수만을 위한 객체 안으로 캡슐화하면 더 유용해지는 상황이 있다.
- 명령 객체 혹은 명령이라 한다.

## 절차

1. 대상 함수의 기능을 옮길 빈 클래스를 만든다. 클래스 이름은 함수 이름에 기초해 짓는다.
2. 방금 생성한 빈 클래스로 함수를 옮긴다. 
→ 리팩터링이 끝날 때까지는 원래 함수를 전달 함수 역할로 남겨주자.
→ 명령 관련 이름은 사용하는 프로그래밍 언어의 명명 규칙을 따른다. 규칙이 딱히 없다면 `execute`나 `call` 같이 명령의 실행 함수에 흔히 쓰이는 이름을 택하자.
3. 함수의 인수들 각각은 명령의 필드로 만들어 생성자를 통해 설정할지 고민해본다. 

# 11.10 명령을 함수로 바꾸기

```jsx
class ChargeCalculator {
	constructor(customer, usage) {
		this._customer = customer;
		this._usage = usage;
	}
	execute() {
		return customer.rate * usage;
	}
}
```

## 다른 예제
- 함수를 한번만 사용할 때 클래스 인스턴스를 생성하는 것은 메모리 낭비
- 차라리 일반함수로 유틸리티 함수로 만드는 것이 더 좋다.
- 프로젝트에서 어떻게 사용할 것인지 목표가 무엇인지에 따라 알맞은 방향으로 리팩터링 해서 개선하자.
```js
export class ChargeCalculator {
  constructor(customer, usage, provider) {
    this._customer = customer;
    this._usage = usage;
    this._provider = provider;
  }
  get baseCharge() {
    return this._customer.baseRate * this._usage;
  }
  get charge() {
    return this.baseCharge + this._provider.connectionCharge;
  }
}

```
```js
export function charge(customer, usage, provider){
	const baseCharge = customer.baseRate * usage;
	return baseCharge + provider.connectionCharge;
}
```
## 배경
- 명령 객체는 복잡한 연산을 다룰 수 있는 강력한 메커니즘을 제공한다.
- 명령의 이런 능력은 공짜가 아니다.
- 그저 함수를 호출하거나 로직이 복잡하지 않으면 장점보다 단점이 크니 평범한 함수로 바꿔주는 게 낫다.

## 절차

1. 명령을 생성하는 코드와 명령의 실행 메서드를 호출하는 코드를 함께 함수로 추출한다.
→ 이 함수가 바로 명령을 대체할 함수다.
2. 명령의 실행 함수가 호출하는 보조 메서드를 각각을 인라인 한다.
→ 보조 메서드가 값을 반환한다면 함수 인라인에 앞서 변수 추출하기를 적용한다.
3. 함수 선언 바꾸기를 적용하여 생성자의 매개변수 모두를 명령의 실행 메서드로 옮긴다.
4. 명령의 실행 메서드에서 참조하는 필드들 대신 대응하는 매개변수를 사용하게끔 바꾼다. 하나씩 수정할 때마다 테스트한다.
5. 생성자 호출과 명령의 실행 메서드 호출을 호출(대체 함수) 안으로 인라인 한다.
6. 테스트한다.
7. 죽은 코드 제거하기로 명령 클래스를 없앤다. 

# 11.11 수정된 값 반환하기

```jsx
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
	for (let i=1; i < points.length; i++) {
		const verticalChange = points[i].elevation - points[i-1].elevation;
		totalAscent += (verticalChange > 0) ? verticalChange : 0;
	}
}
```

```jsx
const totalAscent = calculateAscent();

function calculateAscent() {
	let result = 0;
	for(let i = 1; i < points.length; i++){
		const verticalChange = points[i].elevation - points[i-1].elevation;
		result += (verticalChange > 0) ? verticalChange : 0;
	}
	return result;
}
```
## 다른 예제
- 함수 내부에서 외부에 있는 상태를 변경하거나 인자로 받은 객체를 변경하는 것은 사이드 이팩트를 유발하는 아주 나쁜 함수이다.
- 결과를 리턴, 찾은 값을 리턴, 업데이트 값을 리턴하는 것이 좋다. 
- 최대한 사이드 이팩트를 막는 것이 좋다.
```js
export function ascentVelocity(points, totalMinutes) {
  function calculateAscent() {
    for (let i = 1; i < points.length; i++) {
      const verticalChange = points[i].elevation - points[i - 1].elevation;
      totalAscent += verticalChange > 0 ? verticalChange : 0;
    }
  }

  let totalAscent = 0;
  calculateAscent([{ elevation: 10 }, { elevation: 20 }]);

  return totalAscent / totalMinutes;
}

```
```js
export function ascentVelocity(points, totalMinutes) {
  function calculateAscent() {
    let result = 0;
		for (let i = 1; i < points.length; i++) {
      const verticalChange = points[i].elevation - points[i - 1].elevation;
      result += verticalChange > 0 ? verticalChange : 0;
    }
		return result;
  }

  let totalAscent = calculateAscent([{ elevation: 10 }, { elevation: 20 }]);
  return totalAscent / totalMinutes;
}

```

## 배경

- 데이터가 수정된다면 그 사실을 명확히 알려주어서, 어느 함수가 무슨 일을 하는지 쉽게 알 수 있게 하는 일이 대단히 중요하다.
- 변수를 갱신하는 함수라면 수정된 값을 반환하여 호출자가 그 값을 변수에 담아두도록 하는 것이다.

## 절차

1. 함수가 수정된 값을 반환하게 하여 호출자가 그 값을 자신의 변수에 저장하게 한다.
2. 테스트한다.
3. 피 호출 함수 안에 반환할 값을 가리키는 새로운 변수를 선언한다.
→ 이 작업이 의도대로 이뤄졌는지를 검사하고 싶다면 호출자에서 초깃값을 수정해보자. 제대로 처리했다면 수정된 값이 무시된다.
4. 테스트한다.
5. 계산이 선언과 동시에 이뤄지도록 통합한다(즉, 선언 시점에 계산 로직을 바로 실행해 대입한다).
→ 프로그래밍 언어에서 지원한다면 이 변수를 불편으로 지정하자.
6. 테스트한다.
7. 피 호출 함수의 변수 이름을 새 역할에 어울리도록 바꿔준다.
8. 테스트한다.

# 11.12 오류 코드를 예외로 바꾸기
```jsx
if (data)
	return new ShippingRules(data);
else
	return -23;
```

```jsx
if (data)
	return new ShippingRules(data);
else 
	throw new OrderProcessingError(-23);
```

## 다른 예제
- 에러 클래스를 상속받아서 더 명확한 클래스를 만들어 처리하면 좋다.
```js
function localShippingRules(data) {
  if (data) return new ShippingRules(data);
  else return -23;
}
```
```js
class OrderProcessingError extends Error {
	constructor(errorCode){
		super();
		this.errorCode = errorCode
	}
}	

try {
	const result = localShippingRules();
} catch (error) {
	if(error instanceof OrderProcessingError){
		console.error(error);
	}
}
```

## 배경

- 예외를 사용하면 오류 코드를 일일이 검사하거나 오류를 식별해 콜 스택 위로 던지는 일을 신경 쓰지 않아도 된다.
- 예외는 정교한 메커니즘이지만 대다수의 다른 정교한 메커니즘과 같이 정교하게 사용할 때만 최고의 효과를 낸다.
- 예외는 정확히 예상 밖의 동작일 대만 쓰여야 한다.

## 절차

1. 콜 스택 상위에 해당 예외를 처리할 예외 핸들러를 작성한다.
→ 이 핸들러는 처음에는 모든 예외를 던지게 해둔다.
→ 적절한 처리를 해주는 핸들러가 이미 있다면 지금의 콜스택도 처리할 수 있도록 확장한다.
2. 테스트한다.
3. 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다.
→ 사용하는 프로그래밍 언어에 맞게 선택하면 된다. 대부분 언어에서는 서브 클래스를 사용하면 될 것이다.
4. 정적 검사를 수행한다.
5. `catch`절을 수정하여 직접 처리할 수 있는 예외는 적절히 대처하고 그렇지 않은 예외는 다시 던진다.
6. 테스트한다.
7. 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다. 하나씩 수정할 때마다 테스트한다.
8. 모두 수정했다면 그 오류 코드를 콜 스택 위로 전달하는 코드를 모두 제거한다. 하나씩 수정할 때마다 테스트한다.
→ 먼저 오류 코드를 검사하는 부분을 함정으로 바꾼 다음, 함정에 걸려들지 않는지 테스트한 후 제거하는 전략을 권한다. 함정에 걸려드는 곳이 있다면 오류 코드를 검사하는 코드가 아직 남아 있다는 뜻이다. 함정을 무사히 피했다면 안심하고 본문을 정리하자(죽은 코드 제거하기)

# 11.13 예외를 사전확인으로 바꾸기
```jsx
double getValueForPeriod (int periodNumber){
	try {
		return values[periodNumber];
	} catch (ArrayIndexOutOfBoundsException e) {
		return 0;
	}
}
```
```jsx
double getValueForPeriod (int periodNumber) {
	return (periodNumber >= values.length) ? 0: values[periodNumber];
}
```
## 다른 예제
- 충분히 사전에 예상할 수 있는 오류는 사전에 처리하자.
```js
const values = [];
function getValueForPeriod(periodNumber) {
  const value = values[periodNumber];
  if (!value) {
    throw new Error('value is undefined');
  }
  return value;
}

try {
  getValueForPeriod(-10);
} catch (error) {
  console.log('에러 발생!');
}
```
```js
const values = [];
function getValueForPeriod(periodNumber) {
	if(periodNumber < 0 || periodNumber >= values.length){
		return 0;
	}
	return values[periodNumber];

	// or 
	return values[periodNumber] ?? 0;
}

getValueForPeriod(-10);

```

## 배경

- 함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 검사하도록 해야 한다.

## 절차

1. 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다. catch 블록의 코드를 조건문의 조건절 중 하나도 옮기고, 남은 try 블록의 코드를 다른 조건절로 옮긴다.
2. `catch` 블록에 어서션을 추가하고 테스트한다.
3. `try` 문과 `catch` 블록을 제거한다.
4. 테스트한다.