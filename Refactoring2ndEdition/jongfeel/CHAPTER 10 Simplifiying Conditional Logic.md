## CHAPTER 10 Simplifying Conditional Logic (조건부 로직 간소화)

### 10.1 Decompose Conditional (조건문 분해하기)

``` javascript
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
    charge = quantity * plan.summerRate;
else
    charge = quantity * plan.reqularRate + plan.regularServiceCharge;
```

=>

``` javascript
if (summer())
    charge = summerCharge();
else
    charge = regularCharge();
```

#### 배경

복잡한 조건부 로직은 프로그램을 복잡하게 만드는 가장 흔한 원흉에 속한다. 조건을 검사하고 통과하는 코드는 무슨 일이 일어나는지 알려주지만 '왜'일어나는지 제대로 알려주지 않는다. 

거대한 코드 블록을 부위별로 분해하고 해체한 코드 덩어리들을 의도에 맞는 함수 호출로 바꿔준다.

#### 절차

- 조건식과 그 조건식에 딸리 조건절 각각을 함수호 추출한다.

#### 예시

여름철이면 할인율이 달라지는 어떤 서비스의 요금을 계산한다고 해보자.

### 10.2 Consolidate Conditional Expression (조건식 통합하기)

``` javascript
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDIsabled < 12) return 0;
if (anEmployee.isPartTime) return 0;
```

=>

``` javascript
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisablility() {
    return ((anEmployee.seniority < 2)
        || (anEmployee.monthsDIsabled < 12)
        || (anEmployee.isPartTime));
}
```

#### 배경

비교하는 조건은 다르지만 그 결과로 수행하는 동작은 똑같은 코드는 조건 검사도 하나로 통일한다.

여러 조각으로 나뉜 조건들을 하나로 통합함으로써 내가 하려는 일이 더 명확해진다
이 작업이 함수 추출하기 까지 이어질 가능성이 높다. 함수 추출하기는 무엇을 하는지 보다 왜 하는지를 말해주는 코드로 바꿔주는 효과적인 도구이다.

#### 절차

- 해당 조건식들 모두에 부수효과가 없는지 확인한다.
- 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다
- 테스트한다
- 조건이 하나만 남을 때까지 위 과정을 반복한다.
- 하나로 합쳐진 조건식을 함수로 추출할지 고려해본다.

#### 예시: or 사용하기

#### 예시: and 사용하기

### 10.3 Replace Nested Conditional with Guard Clauses (중첩 조건문을 보호 구문으로 바꾸기)

``` javascript
function getPayAmount() {
    let result;
    if (isDead)
        result = deadAmount();
    else {
        if (isSeparated)
            result = separatedAmount();
        else {
            if (isRetired)
                result = retiredAmount():
            else
                result = normalPayAmount():
        }
    }
    return result;
}
```

=>

``` javascript
function getPayAmount() {
    if (isDead) return deadAmount();
    if (isSeparated) return separatedAmount();
    if (isRetired) return retireAmount();
    return normalPayAmount();
```

#### 배경

조건문은 두 가지 형태로 쓰인다. 참인 경로와 거짓인 경로 모두 정상 동작으로 이어지는 형태와, 한쪽만 정상인 형태다.

중첩 조건문을 보호 구문으로 바꾸기 리팩터링의 핵심은 의도를 부각하는 데 있다.

#### 절차

- 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 바꾼다.
- 테스트한다.
- 위 과정을 필요한 만큼 반복한다.
- 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합한다.

#### 예시

직원 급여를 계산하는 코드 예제, 현직 직원만 급여를 받아야 함.

#### 예시: 조건 반대로 만들기

### 10.4 Replace Conditional with Polymorphism (조건부 로직을 다형성으로 바꾸기)

``` javascript
switch (bird.type) {
    case '유럽 제비':
        return "보통이다";
    case '아프리카 제비':
        return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    case '노르웨이 파랑 앵무':
        return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
    default:
        return "알 수 없다";
```

=> 

``` javascript
class EuropeanSwallow {
    get plumage() {
        return "보통이다";
    }
   ...
class AfricanSwallow {
    get plumage() {
        return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    }
   ...
class NorwegianBlueParrot {
    get plumage() {
        return (this.voltage > 100) ? "그을렸다" : "예쁘다";
    }
```

#### 배경

복잡한 조건부 로직은 프로그래밍에서 해석하기 가장 난해한 대상에 속한다. 조건부 구조를 클래스와 다형성을 이용하면 더 확실하게 분리할 수 있다.

다형성은 객체지향 프로그래밍의 핵심이다. 하지만 남용하기 쉽다. 하지만 개선할 수 있는 조건부 로직을 발견하면 다형성이 막강한 도구임을 깨닫게 된다.

#### 절차

- 다형적 동작을 표현하는 클래스들이 아직 없다면 만들어준다. 이왕이면 적합한 인스턴스를 알아서 만들어 반환하는 팩터리 함수도 함께 만든다.
- 호출하는 코드에서 팩터리 함수를 사용하게 한다.
- 조건부 로직 함수를 슈퍼클래스로 옮긴다.
- 서브클래스 중 하나를 선택한다. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다. 조건부 문장 중 선택된 서브클래스에 해당하는 조건절을 서브클래스 메서드로 복사한 다음 적절히 수정한다.
- 같은 방식으로 각 조건절을 해당 서브클래스에서 메서드로 구현한다.
- 슈퍼클래스 메서드에는 기본 동작 부분만 남긴다. 혹은 슈퍼클래스가 추상 클래스여야 한다면, 이 메서드를 추상으로 선언하거나 서브클래스에서 처리해야 함을 알리는 에러를 던진다.

#### 예시

다양한 새의 종에 따른 비행 속도와 깃털 상태를 알고 싶은 작은 프로그램

#### 예시: 변형 동작을 다형성으로 표현하기

### 10.5 Introduce Special Case

- 1판에서의 이름: Null 검사를 널 객체에 위임

``` javascript
if (aCustomer === "미확인 고객") customerName = "거주자";
```

=>

``` javascript
class UnknownCustomer {
    get name() { return "거주자"; }
}
```

#### 배경

코드베이스에서 특정 값에 대해 똑같이 반응하는 코드가 여러 곳이라면 그 반응을 한 데 모으는게 효율적이다.

Null을 특이 케이스로 처리해야 할 때가 많다. 널 객체 패턴이라고도 하는데 널 외에 다른 특이 케이스에도 적용할 수 있다. 널 객체가 특이 케이스의 특수한 예이다.

#### 절차

속성을 담은 데이터 구조(혹은 클래스)를 컨테이너라고 칭함.
컨테이너를 사용하는 코드에서는 해당 속성이 특이한 값인지를 검사한다.

- 컨테이너에 특이 케이스인지를 검사하는 속성을 추가하고, false를 반환하게 한다.
- 특이 케이스 객체를 만든다. 이 객체는 특이 케이스인지를 검사하는 속성만 포함하며, 이 속성은 true를 반환하게 한다.
- 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출한다. 모든 클라이언트가 값을 직접 비교하는 대신 방금 추출한 함수를 사용하도록 고친다.
- 코드에 새로운 특이 케이스 대상을 추가한다. 함수의 반환 값으로 받거나 변환 함수를 적용하면 된다.
- 특이 케이스를 검사하는 함수 본문을 수정하여 특이 케이스 객체의 속성을 사용하도록 한다.
- 테스트한다.
- 여러 함수를 클래스로 묶기나 여러 함수를 변환 함수로 묶기를 적용하여 특이 케이스를 처리하는 공통 동작을 새로운 요소로 옮긴다.
- 아직도 특이 케이스 검사 함수를 이용하는 곳이 남아 있다면 검사 함수를 인라인한다.

#### 예시

전력 회사는 전력이 필요한 현장(site)에 인프라를 설치해 서비스를 제공한다.

#### 예시: 객체 리터럴 이용하기

데이터 구조를 읽기많 나다면 클래스 대신 리터럴 객체를 사용해도 된다.

#### 예시: 변환 함수 이용하기

### 10.6 Introduce Assertion(어서션 추가하기)

``` javascript
if (this.discountRate)
    base = base - (this.discountRate * base);
```

=>

``` javascript
assert(this.discountRate >= 0);
if (this.discountRate)
    base = base - (this.discountRate * base);
```

#### 배경

어서션은 항상 참이라고 가정하는 조건부 문장으로, 어서션의 실패는 프로그래머가 잘못했다는 뜻이다. 어서션 실패는 시스템의 다른 부분에서는 절대 검사하지 않아야 하며, 어서션이 있고 없고가 프로그램 기능의 정상 동작에 아무런 영향을 주지 않도록 작성돼야 한다.

단위 테스트를 꾸준히 추가하여 사각을 좁히면 어서션 보다 나을 때가 많다. 하지만 다른 프로그래머와 소통 측면에서는 어서션이 여전히 매력적이다.

#### 절차

- 참이라고 가정하는 조건이 보이면 그 조건을 명시하는 어서션을 추가한다.

#### 예시

할인과 관련된 간단한 예, 고객은 상품 구입 시 할인율을 적용받는다.

### 10.7 Replace Control Flag with Break(제어 플래그를 탈출문으로 바꾸기)

- 1판에서의 이름: 제어 플래그 제거

``` javascript
for (const p of people) {
    if (!found) {
        if (p === "조거") {
            sendAlert();
            found = true;
        }
    }
}
...
```

=> 

``` javascript
for (const p of people) {
    if (p === "조거") {
        sendAlert();
        break;
    }
}
...
```

#### 배경

제어 플래그란 코드의 동작을 변경하는 데 사용되는 변수를 말하며, 어딘가에서 값을 계산해 제어 플래그에 설정한 후 다른 어딘가의 조건문에서 검사하는 형태로 쓰인다. 이런 코드는 악취다.

제어 플래그는 반복문에서 나타나는데, 함수의 return 문이 하나여야 하는 것 보다는 함수에서 할 일을 마쳤을 때 return 문으로 명확히 알리는 편이 낫다.

#### 절차

- 제어 플래그를 사용하는 코드를 함수로 추출할지 고려한다.
- 제어 플래그를 갱신하는 코드 각각을 적절한 제어문으로 바꾼다. 하나 바꿀 떄마다 테스트한다.
- 모두 수정했다면 제어 플래그를 제거한다.

#### 예시

사람 목록을 훑으면서 악당(miscreant)을 찾는 코드, 악당 이름은 하드코딩되어 있다.

