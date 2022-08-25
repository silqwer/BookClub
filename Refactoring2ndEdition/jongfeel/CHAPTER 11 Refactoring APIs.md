## CHAPTER 11 Refactoring APIs

### 11.1 Separate Query from Modifier (질의 함수와 변경 함수 분리하기)

``` javascript
function getTotalOutstandingAndSendBill() {
    const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
    sendBill();
    return result;
}
```

=>

``` javascript
function totalOutstanding() {
    customer.invoices.reduce((total, each) => each.amount + total, 0);
}
function sendBill() {
    emailGateway.send(formatBill(customer));
}
```

#### 배경

겉보기 부수효과가 없이 값을 반환하는 함수를 추구해야 한다.

질의 함수는 모두 부수효과가 없어야 한다는 규칙을 따른다.

값을 반환하면서 부수효과도 있는 함수를 발견하면 상태를 변경하는 부분과 질의하는 부분을 분리하려 시도한다. 

#### 절차

- 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다
- 새 질의 함수에서 부수 효과를 모두 제거한다.
- 정적 검사를 수행한다
- 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아낸다. 호출하는 곳에서 반환 값을 사용한다면 질의 함수를 호출하도록 바꾸고, 원래 함수를 호출하는 코드를 바로 아래 줄에 새로 추가한다. 하나 수정할 때마다 테스트 한다.
- 원래 함수에서 질의 관련 코드를 제거한다.
- 테스트한다.

#### 예시

이름 목록을 훑어 악당(miscreant)를 찾는 함수. 악당을 찾으면 그 사람의 이름을 반환하고 경고를 울림. 이 함수는 가장 먼저 찾은 악당만 취급

### 11.2 Parameterize Function (함수 매개변수화하기)

``` javascript
function tenPercentRaise(aPerson) {
    aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
    aPerson.salary = aPerson.salary.multiply(1.05);
}
```

=>

``` javascript
function raise(aPerson, factor) {
    aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

#### 배경

두 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면, 그 다른 값만 매개변수로 받아 처리하는 함수 하나로 합쳐서 중복을 없앨 수 있다.

#### 절차

- 비슷한 함수 중 하나를 선택한다
- 함수 선언 바꾸기로 리터럴들을 매개변수로 추가한다.
- 이 함수룰 호출하는 곳 모두에 적절한 리터럴 값을 추가한다.
- 테스트한다.
- 매개변수로 받으 값을 사용하도록 함수 본문을 수정한다. 하나 수정할 때마다 테스트한다.
- 비슷한 다른 함수를 호출하는 코드를 찾아 매개변수화된 함수를 호출하도록 하나씩 수정한다. 하나 수정할 때마다 테스트한다.

#### 예시

### 11.3 Remove Flag Argument (플래그 인수 제거하기)

1판에서의 이름: 매개변수를 매서드로 전환

``` javascript
function setDimension(name, value) {
    if (name === "height") {
        this._height = value;
        return;
    }
    if (name == "width") {
        this._width = value;
        return;
    }
}
```

=> 

``` javascript
function setHeight(value) { this._height = value; }
function setWidth(value) { this._width = value; }
```

#### 배경

플래그 인수란 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수이다.
플래그 인수가 있는 함수는 호출할 수 있는 함수들이 무엇이고 어떻게 호출해야 하는지 이해하기가 어려워진다.

플래그 인수가 둘 이상이면 함수 하나가 너무 많은 일을 처리하고 있다는 신호이기도 하다. 그러니 같은 로직을 조합해내는 더 간단한 함수를 만들 방법을 고민해야 한다.

#### 절차

- 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성한다.
- 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정한다.

#### 예시

배송일자를 계산하는 호출.

aShipment.deliveryDate = deliveryDate(anOrder, true);

다음 호출로 대체할 수 있다.

aShipment.deliveryDate = rushDeliveryDate(anOrder);

#### 예시: 매개변수를 까다로운 방식으로 사용할 때

### 11.4 Preserve Whole Object (객체 통째로 넘기기)

``` javascript
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))
```

=>

``` javascript
if (aPlan.withinRange(aRoom.daysTempRange))
```

#### 배경

레코드를 통째로 넘기면 변화에 대응하기 쉽다.
그 함수가 더 다양한 데이터를 사용하도록 바뀌어도 매개변수 목록은 수정할 필요가 없다. 그리고 매개변수 목록이 짧아져서 일반적으로는 함수 사용법을 이해하기 쉬워진다.

어떤 객체로 부터 값 몇 개를 얻은 후 그 값들만으로 무언가를 하는 로직이 있다면, 그 로직을 객체 안으로 집어넣어야 함을 알려주는 악취로 봐야 한다.

한 객체가 제공하는 기능 중 항상 똑같은 일부만을 사용하는 코드가 많다면, 그 기능만 따로 묶어서 클래스로 추출하라는 신호일 수도 있다.

#### 절차

- 매개변수들을 원하는 형태로 받는 빈 함수를 만든다.
- 새 함수의 본문에서는 원래 함수를 호출하도록 하며, 새 매개변수와 원래 함수의 매개변수를 매핑한다.
- 정적 검사를 수행한다.
- 모든 호출자가 새 함수를 사용하게 수정한다. 하나씩 수정하며 테스트하자
- 호출자를 모두 수정했다면 원래 함수를 인라인한다
- 새 함수의 이름을 적절히 수정하고 모든 호출자에 반영한다.

#### 예시

실내온도 모니터링 시스템, 이 시스템은 일일 최저, 최고 기온이 난반 계획(heating plan)에서 정한 범위를 벗어나는지 확인한다.

#### 예시: 새 함수를 다른 방식으로 만들기

### 11.5 Replace Parameter with Query (매개변수를 질의 함수로 바꾸기)

- 반대 리팩터링: 질의 함수를 매개변수로 바꾸기
- 1판에서의 이름: 매개변수 세트를 매서드로 전환

``` javascript
availableVacation(anEmployee, anEmployee.grade);

function avaliableVacation(anEmployee, grade {
    // 연휴 계산...
```

=>

``` javascript
avaliableVacation(anEmployee)

function avaliableVacation(anEmployee) {
    const grade = anEmployee.grade;
    // 연휴 계산...
```

#### 배경

매개변수는 함수의 변동 요인이다. 함수에 변화를 줄 수 있는 일차적인 수단이다.

매개변수가 있다면 결정 주체자가 호출자가 되고,
매개변수가 없다면 피호출 함수가 된다.

매개변수를 제거하면 피호출 함수에 원치 않은 의존성이 생긴다. 해당 함수가 알지 못했으면 하는  프로그램 요소에 접근헤야 하는 상황을 만들 때다.

#### 절차

- 필요하다면 대상 매개변수의 값을 계산하는 코드를 별도 함수로 추출해놓는다.
- 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다. 하나 수정할 때마다 테스트한다.
- 함수 선언 바꾸기로 대상 매개변수를 없앤다.

#### 예시

다른 리팩터링을 수행한 뒤 특정 매개변수가 더는 필요 없어졌을 때

### 11.6 Replace Query with Parameer (질의 함수를 매개 변수로 바꾸기)

- 반대 리팩터링: 매개 변수를 질의 함수로 바꾸기

``` javascript
targetTemperature(aPlan)

function targetTemperature(aPlan) {
    currentTemperature = thermostat.currentTemperature;
    // 생략
```

=>

``` javascript
targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
    // 생략
```

#### 배경

코드를 읽다 보면 함수 안에 두기엔 거북한 참조를 발견할 때가 있다. 전역 변수를 참조한다거나 제거하길 원하는 원소를 참조하는 경우가 여기 속한다.

이런 상황 대부분은 코드의 의존 관계를 바꾸려 할 때 벌어진다. 대상 함수가 더 이상 (매개변수화하려는) 특정 원소에 의존하길 원치 않을 떄 일어난다. 한 시점에 내린 결정이 영원히 옳다고 할 수 없는 문제다. 따라서 프로그램을 더 잘 이해하게 됐을 때 더 나은 쪽으로 개선하기 쉽게 설계해 두어야 한다.

참조 투명하지 않은 원소에 접근한느 모든 함수는 참조 투명성을 잃게 되는데, 이 문제는 해당 원소를 매개변수로 바꾸면 해결된다. 모듈을 참조 투명하게 만들어 얻는 장점은 대체로 아주 크다.

질의 함수를 매개 변수로 바꾸면 호출자가 복잡해지는데, 복잡도의 단순함을 호출자로 두느냐 피호출자로 두느냐 즉 책임 소재를 프로그램의 어디에 배정하느냐의 문제로 귀결된다. 프로젝트를 진행하면서 균형점이 이러저리 옮겨질 수 있으니 이 리팩터링과 그 반대 리팩터링과는 아주 친해져야 한다.

#### 절차

- 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리한다.
- 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출한다.
- 방금 만든 변수를 인라인하여 제거한다.
- 원래 함수도 인라인한다.
- 새 함수의 이름을 원래 함수의 이름으로 고쳐준다.

#### 예시

실내온도 제어 시스템, 사용자는 온도조절기(thermostat)로 온도를 설정할 수 있지만, 목표 온도는 난방 계획에서 정한 범위에서만 선택할 수 있다.

### 11.7 Remove Setting Method (세터 제거하기)

``` javascript
class Person {
    get name() { ... }
    set name(aString) { ... }
}
```

=>

``` javascript
class Person {
    get name() { ... }
}
```

#### 배경

이 리팩터링이 필요한 상황은 아래 두 가지다.

- 사람들이 무조건 접근자 메서드를 통해서만 필드를 다루려 할 때.
- 클라리언트에서 생성 스크립트(creation script)를 사용해 객체를 생성할 때. 생성 스크립트는 생성자를 호출한 후에 세터들을 호출하려 객체를 완성하는 형태의 코드이다.

#### 절차

- 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다. 그런 다음 생성자 안에서 적절한 세터를 호출한다.
- 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 때마다 테스트한다.
- 세터 메서드를 인라인한다. 가능하다면 해당 필드를 불변으로 만든다.
- 테스트한다.

#### 예시

간단한 사람(Person) 클래스에서 id 값에 해당하는 세터를 제거하고 생성자로 대체

### 11.8 Replace Constructor with Factory Function (생성자를 팩터리 함수로 바꾸기)

1판에서의 이름: 생성자를 팩토리 메서드로 전환

``` javascript
leadEngineer = new Employee(document.leadEngineer, 'E');
```

=>

``` javascript
leadEngineer = createEngineer(document.leadEngineer);
```

#### 배경

생성자에는 제약이 따르는데
자바에서는 생성자를 정의한 클래스의 인스턴스를 반환해야 하고
생성자의 이름도 고정되어 있다.
생성자를 호출하려면 new 라는 키워드를 사용해야 한다.

팩터리 함수를 사용하면 이런 제약을 없앨 수 있다.

#### 절차

- 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래의 생성자를 호출한다.
- 생성자를 호출하던 코드를 팩터리 함수 호출로 바꾼다.
- 하나씩 수정할 때마다 테스트한다.
- 생성자의 가시 범위가 최소가 되도록 제한한다.

#### 에시

직원(Employee) 유형을 다루는, 간단하지만 이상한 예.

### 11.9 Replace Function with Command (함수를 명령으로 바꾸기)

반대 리팩터링: 명령을 함수로 바꾸기
1판에서의 이름: 메서드를 메서드 객체로 전환

``` javascript
function score(candidate, medicalExam, scoringGuide) {
    let result = 0;
    let healthLevel = 0;
    // 긴 코드 생략
}
```

=>

``` javascript
class Scorer {
    constructor(candidate, medicalExam, scoringGuide) {
        this._candidate = candidate;
        this._medicalExam = medicalExam;
        this._scoringGuide = scoringGuide;
    }
    execute() {
        this._result = 0;
        this._healthLevel = 0;
        // 긴 코드 생략
    }
}
```

#### 배경

함수를 그 함수만을 위한 객체 안으로 캡슐화하면 더 유용해지는 상황이 있다. 이런 객체를 '명령 객체' 혹은 단순히 '명령(command)' 이라 한다. 이 메서드를 요청해 실행하는 것이 이 객체의 목적이다.

명령은 평범한 함수 매커니즘보다 훨씬 유연하게 함수를 제어하고 표현할 수 있다. 이런 유연성은 복잡성을 키우고 얻는 대가임을 잊지 말아야 한다.

#### 절차

- 대상 함수의 기능을 옮길 빈 클래스를 만든다. 클래스 이름은 함수 이름에 기초해 짓는다.
- 방금 생성한 빈 클래스로 함수를 옮긴다.
- 함수의 인수들 각각은 명령의 필드로 만들어 생성자를 통해 설정할지 고민해본다.

#### 예시

복잡한 함수를 잘게 쪼개서 이해하거나 수정하기 쉽게 만들때, 건강보험 어플리케이션에서 사용하는 점수 계산 함수.

### 11.10 Replace Command with Function (명령을 함수로 바꾸기)

반대 리팩터링: 함수를 명령으로 바꾸기

``` javascript
class ChargeCalculator {
    constructor(customer, usage) {
        this._customer = customer;
        this._usage = usage;
    }
    execuate() {
        return this._customer.rate * this._usage;
    }
}
```

=>

``` javascript
function charge(customer, usage) {
    return customer.rate * usage;
}
```

#### 배경

함수를 하나 호출해 정해진 일을 수행하는 용도로 명령(command)을 사용하는데
로직이 크게 복잡하지 않다면 명령 객체는 장점보다 단점이 크기 평범한 함수로 바꿔주는 게 낫다.

#### 절차

- 명령을 생성하는 코드의 명령의 실행 메서드를 호출하는 코드를 함께 함수로 추출한다.
- 명령의 실행 함수가 호출하는 보조 메서드들 각각을 인라인한다.
- 함수 선언 바꾸기를 적용하여 생성자의 매개변수 모두를 명령의 실행 매서드로 옮긴다.
- 명령의 실행 메서드에서 참조하는 필드들 대신 대응하는 매개변수를 사용하게끔 바꾼다. 하나씩 수정할 때마다 테스트한다.
- 생성자 호출과 명령의 실행 메서드 호출을 호출자(대체 함수) 안으로 인라인한다.
- 테스트한다.
- 죽은 코드 제거하기로 명령 클래스를 없앤다.

### 11.11 Return Modified Value (수정된 값 반환하기)

``` javascript
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
    for (let i = 0; i < points.length; i++) {
        const verticalChange = points[i].elevation - points[i-1].elevation;
        totalAscent += (verticalChange > 0) ? verticalChange : 0;
    }
}
```

=>

``` javascript
const totalAscent = calculateAscent();

function calculateAscent() {
    let result = 0;
    for (let i = 1; i < points.length; i++) {
        const verticalChange = points[i].elevation - points[i-1].elevation;
        result += (verticalChange > 0) ? verticalChange : 0;
    }
    return result;
}
```

#### 배경

데이터가 어떻게 수정되는지를 추적하는 일은 코드에서 이해하기 가장 어려운 부분 중 하나다. 데이터가 수정된다면 그 가실을 명확히 알려주어서 어느 함수가 무슨 일을 하는지 쉽게 알 수 있게 하는 일이 대단히 중요하다.

데이터가 수정됨을 알려주는 좋은 방법은 수정된 값을 반환하여 호출자가 그 값을 변수에 담아두도록 하는 것이다.

#### 절차

- 함수가 수정된 값을 반환하게 하여 호출자가 그 값을 자신의 변수에 저장하게 한다.
- 테스트한다.
- 피호출 함수 안에 반환할 값을 가리키는 새로운 변수를 선언한다.
- 테스트한다.
- 계산이 선언과 동시에 이뤄지도록 통합한다. (즉, 선언 시점에 계산 로직을 바로 실행해 대입한다)
- 테스트한다.
- 피호출 함수의 변수 이름을 새 역할에 어울리도록 바꿔준다.
- 테스트한다.

#### 예시

GPS 위치 목록으로 다양한 계산을 수행하는 코드.

### 11.12 Replace Error Code with Exception (오류 코드를 예외로 바꾸기)

``` javascript
if (data)
    return new ShippingRules(data);
else
    return -23;
```

=>

``` javascript
if (data)
    return new ShippingRules(data);
else
    throw new OrderProcessingError(-23);
```

#### 배경

예외는 프로그래밍 언어에서 제공하는 독릭접인 오류 처리 매커니즘이다. 오류가 발견되면 예외를 던진다.

예외는 정확히 예상 밖의 동작일 때만 쓰여야 한다. 달리 말하면 프로그램의 정상 동작 범주에 들지 않는 오류를 나타낼 때만 쓰여야 한다.

#### 절차

- 콜스택 상위에 해당 예외를 처리할 핸들러를 작성한다.
- 테스트한다.
- 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다.
- 정적 검사를 수행한다.
- catch절을 수정하여 직접 처리할 수 있는 예외는 적절히 대체하고 그렇지 않은 예외는 다시 던진다.
- 테스트한다.
- 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다. 하나씩 수정할 때마다 테스트한다.
- 모두 수정했다면 그 오류 코드를 콜스택 위로 전달하는 코드를 모두 제거한다. 하나씩 수정할 때마다 테스트한다.

#### 에시

전역 테이블에서 배송지의 배송 규칙을 알아내는 코드.

### 11.13 Replace Exception with Precheck (예외를 사전확인으로 바꾸기)

1판에서의 이름: 예외 처리를 테스트로 교체

``` javascript
double getValueForPeriod (int periodNumber) {
    try {
        return values[periodNumber];
    } catch (ArrayINdexOutOfBoundsException e) {
        return 0;
    }
}
```

=>

``` javascript
double getValueForPeroid (int peroidNumber) {
    return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```

#### 배경

예외는 '뜻박의 오류'라는, 말 그대로 예외적으로 동작할 때만 쓰여야 한다. 함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 검사하도록 해야 한다.

#### 절차

- 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다. catch 블록의 코드를 조건문의 조건절 중 하나로 옮기고, 남은 try 블록의 코드를 다른 조건절로 옮긴다.
- catch 블록에 어서션을 추가하고 테스트한다.
- try문과 catch 블록을 제거한다.
- 테스트한다.

#### 예시 (자바)

데이터베이스 연결 같은 자원들을 관리하는 자원 풀(resource pool) 클래스가 있다고 해보자. 자원이 필요한 코드는 풀에서 하나씩 꺼내 사용한다. 풀은 어떤 자원이 할당되었고 가용한 지를 추적하고, 자원이 바닥나면 새로 생성한다.