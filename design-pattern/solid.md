# SOLID 원칙

소프트웨어 구조가 변경에 유연하고, 이해하기 쉽도록 작성하기 위한 목적을 가지고 있는 5개의 원칙

* **S**ingle Responsibility Principle
* **O**pen Close Principle
* **L**iscov Substitution Principle
* **I**nterface Segregation Principle
* **D**ependency Inversion Principle\


## 단일 책임 원칙 SRP Single Responsibility Principle

> There should never be more than one reason for a class to change.

### 정의

작성된 클래스는 하나의 기능만을 가지며 클래스가 제공하는 모든 서비스는 그 하나의 책임을 수행하는 데 집중해야 한다 (응집성)

클래스는 그 책임을 완전히 캡슐화 해야 한다

클래스가 제공하는 모든 기능은 이 책임과 주의깊게 부합해야 한다

### 장점

* 책임 영역이 분명해지기 때문에 해당 책임이 변경되었을 때 영향 범위가 다른 책임에 영향을 끼치지 않음
* 책임을 적절히 분리 -> 가독성 향상, 유지보수 용이



## 개방 폐쇄 원칙 OCP Open Close Principle

> You should be able to extend a classes behavior, without modifying it.

### 정의

소프트웨어의 구성요소(컴포넌트, 클래스, 모듈, 함수)는 확장에는 열려있고, 변경에는 닫혀있어야 한다

즉, 기존 구성요소를 수정하지 않으면서 새로운 요구사항이나 변화를 적용할 수 있도록 해야 한다 -> 추상화, 다형성

### 장점

기능을 추가하거나 변경해야 할 때 기존의 코드를 수정하지 않고도 추가가 가능해진다

### 주의

* 적절한 조절이 필요😅 그렇지 않으면 오히려 더 복잡해질 수 있다!!
  * 확장되는 것과 변경되지 않는 모듈을 잘 구분해야 함
  * 인터페이스 설계 시 추상화 레벨 & 정의하는 범위를 잘 조절해야 함 (인터페이스는 가능한 변경하지 않아야 함)

\


## 리스코브 치환 원칙 LSP The Liskov Substitution Principle

> Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.



### 정의

서브 타입은 언제나 기반 타입으로 교체할 수 있어야 한다 (자식 클래스는 부모 클래스의 역할을 대신할 수 있어야 한다)

상위 타입 객체를 하위 타입 객체로 치환해도 정상적으로 동작해야 한다

상속 관계에서 IS-A 관계가 성립해야 한다 (일관성이 있어야 한다)



> 자료형 S 가 자료형 T의 하위형이면, 프로그램에서 자료형 T의 객체는 프로그램의 속성을 변경하지 않고 자료형 S의 객체로 교체할 수 있다



### 위반 예시 1

부모클래스 : 도형

자식 클래스 : 사각형, 원

도형클래스가 아래와 같은 기능을 수행한다고 가정했을 때, 도형 클래스는 사각형 클래스와 IS-A(일반화)관계가 성립하나 원 클래스와는 성립하지 않는다.&#x20;

1. 도형은 둘레를 가지고 있다
2. 도형은 넓이를 가지고 있다
3. 도형은 각을 가지고 있다 (원의 경우 “원은 각을 가지고 있다”라는 부분이 어색, LSP를 만족하지 않음)

### 위반 예시 2

부모 클래스 : 직사각형

자식 클래스 : 정사각형

직사각형은 높이와 너비가 독립적으로 변경될 수 있지만, 정사각형은 높이와 너비가 같아야 한다. 자식인 정사각형이 부모인 직사각형을 대체할 수 없으므로 LSP 법칙을 위반한다

\


## 인터페이스 분리 원칙 Interface Segregation Principle

> Clients should not be forced to depend upon interfaces, that they do not use.

### 정의

클래스는 자신이 사용하지 않는 인터페이스는 구현하지 않아야 한다&#x20;

자신이 이용하지 않는 메서드에 의존하지 않아야 한다

즉, 하나의 일반적인 인터페이스보다는 여러 개의 구체적인 인터페이스가 낫다 (인터페이스의 단일 책임)\


### 장점

작은 단위로 분리하여 꼭 필요한 메서드들만 이용 -> 의존성을 낮춰서 리팩토링, 수정에 용이한 구조가 됨

사용하지 않는 인터페이스의 변경이 발생했을 때, 영향을 받을 일이 없어짐

\


## 의존성 역전 원칙 DIP Dependency Inversion Principle

> High level modules should not depend upon low level modules. Both should depend upon abstractions.

> Abstractions should not depend upon details. Details should depend upon abstractions.

### 정의

상위 계층(정책 결정)이 하위 계층(세부 사항)에 의존하는 관계를 역전, 상위 계층이 하위 계층의 구현으로부터 독립되어야 한다

의존 관계를 맺을 때 변화하기 쉬운 것보다 변화하기 어려운 것에 의존해야 한다

1. 상위 모듈은 하위 모듈에 의존해서는 안된다. 상위와 하위 객체 모두가 동일한 추상화에 의존해야 한다.
2. 추상화는 세부 사항에 의존해서는 안된다. 세부사항이 추상화에 의존해야 한다.\


### 장점

변동성이 큰 구현체에 의존하지 않고 안정된 추상 인터페이스에 의존 -> 보다 안정적인 프로그램 작성 가능

<figure><img src="../.gitbook/assets/스크린샷 2023-01-09 오후 3.01.29.png" alt=""><figcaption></figcaption></figure>

### 적용 예시

* 내부에서 직접 선언으로 객체 생성 금지, 대신 추상 팩토리 사용하기
* 상속은 신중하게 사용하기
* 구체 함수를 오버라이딩하지 않기, 대신 추상 함수로 선언하고 각 구현체들이 각자 용도에 맞게 구현하도록 하기\


### 예외

변경 가능성이 거의 없는 클래스(ex.String)나 운영체제, 플랫폼처럼 안정성이 보장된 환경에서는 해당 법칙을 무시한다

\
